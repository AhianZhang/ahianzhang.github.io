---
title: "JVM 常用参数查看"
date: "2018-09-02 16:57:36"
tags: ["性能调优"]
categories: ["架构"]
---
### JVM 参数
- 标准参数
- -X 参数
- -XX 参数  

#### 标准参数
```bash
-help
-server
-client
-version 
-showversion
-cp
-classpath
...
```
#### X 参数
```bash
-Xint:解释执行
-Xcomp：第一次使用就编译成本地代码(速度比慢)
-Xmixed：混合模式，JVM 自己决定是否编译成本地代码
```
例如：
```
java -Xint - version
----------------------
output:
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, interpreted mode)
```
#### XX 参数
##### Boolean类型
```bash
 -XX:[+-]<name> #表示启用或者禁用 name 属性
```
举例：
```bash
-XX:+UserConcMarkSweepGC #启用 CMS 垃圾收集器
-XX:+UserG1GC #启用了 G1 垃圾收集器
```
##### Key-Value 类型
```bash
-XX:<name>=<value> #表示 name 属性的值是 value
```
举例：
```bash
-XX:MaxGCPauseMillis=500 #最大停顿时间
-XX:GCTimeRatio=19 
```

##### -Xmx(最大内存) -Xms(最小内存) 
-Xmx <=> -XX:MaxHeapSize
-Xms <=> -XX:InitialHeapSize

#### JVM 运行时参数
```bash
-XX:+PrintFlagsInitial #查看初始值
-XX:+PrintFlagsFinal #查看最终值
-XX:+UnlockExperimentalVMOptions #解锁实验性参数
-XX:+UnlockDiagnosticVMOptions #解锁诊断参数
-XX:+PrintCommandLineFlags #打印命令行参数
```
结果中的 ``=`` 表示默认值； ``:=`` 被用户修改后的 JVM 的值。

### 查看参数的指令

#### jps 查看 Java 进程
使用 ``jps -l``命令查看进程号和具体的 Java 程序
#### jinfo 查看进程内的某个参数的值
```bash
jinfo -flag <name> pid
jinfo -flags <name> pid #打印非默认值
```
#### jstat 查看 JVM 统计信息
类加载、垃圾收集、JIT 编译
```
-class
-gc ...
-compiler
```
```bash
jstat -gc 32144 1000 10 #每隔1000ms打印一次，打印十次
```

-gc输出的结果
- S0C S1C S0U S1U : S0 和 S1 的总量和使用量
- EC EU ： Eden 区总量与使用量
- OC OU ： Old 区总量与使用量
- MC MU ： Metaspace 区总量与使用量
- CCSC CCSU ： 压缩类空间总量与使用量
- YGC YGCT ：YoungGC 的次数与时间
- FGC FGCT ： FullGC 的次数与时间
- GCT ：总的 GC 时间  
  