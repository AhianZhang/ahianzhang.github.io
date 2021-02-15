---
title: "jmap & mat 内存溢出"
date: "2018-09-02 19:46:08"
tags: ["性能调优"]
categories: ["架构"]
---
### 模拟内存溢出
![jvm](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-07-08-031040.png)
 S0 和 S1 同时只会有一个使用，另一个是空的。
Metaspace 中主要存放 class 和 methods 等

模拟溢出的环境
堆内存溢出：通过不断地创建对象来将堆内存填充满
非堆内存溢出：通过继承 ClassLoader 配合 asm 工具来动态的创建类，直到将非堆内存填充满。这个代码可以[点击这里查看](https://gist.github.com/AhianZhang/c4c2c4ee6fb7a82190ac157c3bc606db)

将启动的 VM 参数调小，方便快速实现效果。

```java
@RestController
public class MemoryController {
	
	private List<User>  userList = new ArrayList<User>();
	private List<Class<?>>  classList = new ArrayList<Class<?>>();
	
	/**
	 * -Xmx32M -Xms32M
	 * */
	@GetMapping("/heap")
	public String heap() {
		int i=0;
		while(true) {
			userList.add(new User(i++, UUID.randomUUID().toString()));
		}
	}
	
	
	/**
	 * -XX:MetaspaceSize=32M -XX:MaxMetaspaceSize=32M
	 * */
	@GetMapping("/nonheap")
	public String nonheap() {
		while(true) {
			classList.addAll(Metaspace.createClasses());
		}
	}
	
}

```
输出的异常分别为：
```
Exception in thread "http-nio-8080-exec-2" java.lang.OutOfMemoryError: GC overhead limit exceeded
```
```
Exception in thread "http-nio-8080-exec-2" Exception in thread "http-nio-8080-exec-1" java.lang.OutOfMemoryError: Metaspace
java.lang.OutOfMemoryError: Metaspace
Exception in thread "ContainerBackgroundProcessor[StandardEngine[Tomcat]]" java.lang.OutOfMemoryError: Metaspace
```
### 导出内存映像文件
C++ 内存泄漏是指 new 一个对象后指针丢失了，导致整个对象无法被回收，Java是指 new 了以后一直被占有没被释放。

#### 内存溢出自动导出
```
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./
```
#### 使用 jmap 命令手动导出
```
jmap -dump:format=b,file=heap.hprof pid
```
有时内存大的时候有可能无法自定导出，这是就需要使用 jmap 手动导出

#### 使用 mat 工具进行分析
mat 是 eclipse 基金会下面的一款内存分析工具。可以从官网上下载。
运行 mat.exe 导入之前的 heap.hprof 文件，首页的 overview 一目了然不用过多解释，下面是常用的操作

![mat](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-07-08-031055.png) 

在需要查看的对象上右键 merge shortest path to gc roots->exclude all phantom/week/soft,etc. reference
去掉所有的虚引用只留下强引用，此时会很容易排查.


通过上面的方式基本上就能定位到致内存溢出的对象。
