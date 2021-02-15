---
title: "Tomcat 远程 Debug"
date: "2018-09-04 23:41:22"
tags: ["性能调优"]
categories: ["架构"]
---
### JDWP 协议介绍
JDWP 是 Java Debug Wire Protocol 的缩写，它定义了调试器（debugger）和被调试的 Java 虚拟机（target vm）之间的通信协议。
详请参考[此文](https://www.ibm.com/developerworks/cn/java/j-lo-jpda3/)。

### 远程 Tomcat 服务器配置
#### 修改启动文件
```bash
vi bin/startup.sh 
```
在启动命令中添加启动参数 jpda
```
exec "$PRGDIR"/"$EXECTABLE" jpda start "$@"
```

#### 修改 catalina 文件（主要配置）
```bash
vi bin/catalina.sh
```
```bash
if [ "$1" = "jpda" ] ; then
  if [ -z "$JPDA_TRANSPORT" ]; then
    JPDA_TRANSPORT="dt_socket"
  fi
  if [ -z "$JPDA_ADDRESS" ]; then
    JPDA_ADDRESS="localhost:8000"//这里是要修改的
  fi
  if [ -z "$JPDA_SUSPEND" ]; then
    JPDA_SUSPEND="n"
  fi
  if [ -z "$JPDA_OPTS" ]; then
    JPDA_OPTS="-agentlib:jdwp=transport=$JPDA_TRANSPORT,address=$JPDA_ADDRESS,server=y,suspend=$JPDA_SUSPEND"
  fi
  CATALINA_OPTS="$JPDA_OPTS $CATALINA_OPTS"
  shift
fi
```
done
### 项目代码配置
将项目文件打成 War 包 ， 如果是 SpringBoot 项目的话可以使用下面的打包方式：
1、Pom 文件将打包方式改成 war 启动类
2、启动类修改
```java
@SpringBootApplication
public class TestApplication extends SpringBootServletInitializer{
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(TestApplication.class);
    }


    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}
```
3、mvn build/package

### 编辑器远程调试
- Eclipse：Debug Configurations->Remote Java Application
- IDEA：Run -> Edit Configuration -> add -> remote

配置好 host 和 port 就可以打断点进行调试了。
