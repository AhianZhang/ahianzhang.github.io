---
title: "AMQ Model总结"
date: "2020-03-21 16:38:14"
tags: ["rabbitmq"]
categories: ["协议"]
---



# 前言

最近在写公司的消息队列组件，因为使用的是 RabbitMQ，其实现的规范是基于 AMQP-0-9-1 ，所以抽时间把官方的规范过了一遍，整理出主要的内容，记录于此。

# AMQ 模型架构(**AMQ Model Architecture**)

## 基础组件

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-03-21-091225.png)

- Publisher application 发送者
- Consumer application 消费者
- Server AMQP 服务器
- Virtual host 在 Server 中提供逻辑隔离，类似于 MySQL 数据库中的多个表
- Exchange 将消息根据不同的配置路由到不同队列
- Message Queue 存储消息的地方

## 消息流转

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-03-21-093617.png)

从上图可看出，AMQP 的关键之处和难点在于 Exchange 是如何将不同的消息发送到不同的 Message Queue 中

## 消息的生命周期

在发送者发送消息和消费者确认消费，对于如何保证消息的可靠性，在 RabbitMQ 中有两种方式，一种是放到同一个事务中处理，但是这种方式对于并发量不大的可以使用，保证了准确性，但是牺牲了性能；还有一种是轻量级的通过 confirm 去异步确认，这部分以后会单独写出来。

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-03-22-135505.png)

## 消息队列

消息队列能将消息存到内存或者磁盘上，供消费者消费。它有几个主要的配置

- 名称 name
- 专用 exclusive： 随 channel 创建和销毁
- 持久化 durable：broker 重启后能够恢复持久化的消息

### 队列生命周期

- 持久化队列
- 临时队列

![临时队列生命周期](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-03-22-140750.png)

## 交换机

交换机，根据预先设定的 bindings 规范进行匹配和路由，exchange 根据 bindings 的信息决定应该将消息发送到哪个 queue 或者 exchange，exchange 此模块不存储任何消息，对于使用时我们一般说的是交换机的类型，就像车，有火车、汽车、自行车等，而 exchange 有四种类型，下面介绍常用的三种。

### direct

AMQP 默认使用的是 direct 模式，此模式的特点是 routing key 和 bindings 必须匹配才能发送成功,如下图所示，如果 routing key 为 abc 的消息只能被 Consumer A 消费。

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-03-22-122824.png)

### topic

topic 模式可以说是 direct 的一种扩展，这个扩展支持使用通配符来进行路由，规则是 **#** 代表多个单词，***** 代表单个单词，下图中的消息能够被 Consumer A 和 Consumer C 消费，而 Consumer B 所监听的队列绑定的是 a.* ，最多只能是两个字母，不匹配。

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-03-22-123935.png)

### fanout

此模式也能叫做广播模式，此模式没有 routing key 的概念，发送方将消息发到 exchange 中，只要是有消息队列与之绑定就直接发送过去。

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-03-22-124807.png)

### exchange 生命周期

在 borker 启动时会创建内部的交换机，这些交换机无法删除，还有一种就是应用创建（官方觉得叫 声明 更好），声明一个私有的交换机，私有的交换机可以被删除，但是在实际使用时声明出来以后不会再删。

## Routing Key

存在于发送者和 broker 之间，交换机会根据 routing key 将消息发送到对应的队列中。

## Bindings

Bindings 存在于 exchange 和 message queue 之间，用来告诉 exchange 应该将消息路由到哪个队列上。

利用 bindings 可以实现

- 共享队列(Shared Queue)：多个消费者监听同一个队列
- 应答队列(Reply Queue)：应用不声明队列，由 broker 临时生成一个队列进行消费
- 发布订阅队列(Pub/Sub Queue)：使用 topic 类型实现，结合 ``bind()`` 方法

---

除了以上的 AMQ 模型外，AMQP 还有命令、传输、功能、技术架构，这些等有时间再看。



