---
title: "使用 psi-probe 监控 Tomcat"
date: "2018-09-06 23:51:38"
tags: ["性能调优"]
categories: ["架构"]
---
psi-probe 是一款非常强大的开源 Tomcat 监控工具，使用时可以点击[这里](https://github.com/psi-probe/psi-probe/releases)下载

配置用户：

在 tomcat/webapps/docs/deployer-howto.html 文档的 manager 下有说明

```
conf/tomcat-user.xml
```
1、添加用户
```
<role rolename="ahian"/>
<role rolename="manager-gui"/>
<role rolename="manager-status"/>
<user username="ahian" password="123456" roles="ahian,manager-gui,manager-status"/>
```
2、在 conf 文件夹中新建 Catalina/localhost/manager.xml 并编辑
```xml
<?xml version="1.0" encoding="UTF-8"?>
	<Context privileged="true"
	  antiResourceLocking="false"
	  docBase="${catalina.home}/webapps/manager"
		>
	         <Valve className="org.apache.catalina.valves.RemoteAddrValve"
	                allow="127\.0\.0\.1"/>
	</Context>
```
3、将 probe.war 包拷贝到 webapp 下，启动 tomcat 
4、访问 localhost:8080/probe