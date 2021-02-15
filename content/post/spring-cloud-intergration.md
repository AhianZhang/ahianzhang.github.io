---
title: "微服务应用集成 SpringCloud 步骤"
date: "2020-10-13 15:14:02"
tags: ["SpringCloud"]
categories: ["总结"]
---

# 前言

微服务开发模式能够将复杂的业务拆分成独立的更细粒度服务，一定程度上降低了业务复杂度，但是随之而来的是如何保证各个服务的可靠性和稳定性，SpringCloud 出现的目的是为了解决这些在服务拆分后会遇到的通用问题。配置中心能让同一服务多实例间无需重启快速更新配置、注册中心能对应用节点动态管理、网关是为了对 API 做统一的处理等。

# 步骤

## 服务发现

**默认有已经部署好了 Nacos Server。**

引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>${latest.version}</version>
</dependency>
```

创建 bootstrap.yml 并添加下面的配置

```yaml
discovery:
  group: infra
  namespace: 7925f1c5-3291-4c0b-b4c4-920a532958f1
  server-addr: 192.168.110.65:8848
```

## 服务配置

**默认有已经部署好了 Nacos Server。**

引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>${latest.version}</version>
</dependency>
```

在 bootstrap.yml 中添加下面的配置

```yaml
nacos:
  config:
    file-extension: yaml
    group: INFRA
    namespace: 7925f1c5-3291-4c0b-b4c4-920a532958f1
    server-addr: 192.168.110.65:8848
```

配置完成后 bootstarp.yml 的样例：

```yaml
spring:
  application:
    name: auth-service
  cloud:
    nacos:
      config:
        file-extension: yaml
        group: INFRA
        namespace: 7925f1c5-3291-4c0b-b4c4-920a532958f1
        server-addr: 192.168.110.65:8848
      discovery:
        group: infra
        namespace: 7925f1c5-3291-4c0b-b4c4-920a532958f1
        server-addr: 192.168.110.65:8848
```

## API 网关

**默认 API 网关以部署好并与 nacos 做好了集成**，**对外提供动态路由配置、认证、限流等功能。**以 Spring Cloud Gateway 为例。

在 API Gateway 的配置中心中添加路由配置，示例：

```json
{
    "id":"oauth-test", 
    "predicates":[
        {
            "name":"Path",
            "args":{
                "pattern":"/baidu/**"
            }
        }
    ],
    "uri":"http://baidu.com/",
    "filters":[
        {
            "name":"StripPrefix",
            "args":{
                "parts":"1"
            }
        },
        {
            "name":"Auth"
        }
    ]
}
```

说明：

* id：路由的唯一标识
* predicates：前置判断，符合相应的配置才能交给后面的过滤器
* uri：最终调整的地址
* filters：过滤器集合，这里配了 StripPrefix（跳转后会抹掉 /baidu 这个路径），Auth（开启认证过滤）



## 监控

计划中，Prometheus + Grafana 

## 链路追踪

计划中，Spring Cloud Sle	uth + ZipKin 或者 CAT

## 熔断

计划中，Spring Cloud Alibaba Sentinel

## 分布式事务（可选）

目前在 Spring 官方文档中并未将事务纳入 Cloud 中，但是在实际开发中如果需要也可使用。

计划中，Spring Cloud Alibaba Seata

# 总结

SpringCloud 设计初衷是为了解决微服务治理相关的问题，提供一套开箱即用的组件，让开发者将重点放到服务拆分和设计上。上面介绍的几个组件都是微服务治理体系中必不可少的内容，所以如果想从单体应用转到微服务一定要搭建相应的治理平台，才能保证服务的可靠性和稳定性。