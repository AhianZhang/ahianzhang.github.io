---
title: "RSocket 介绍"
date: "2019-12-10 14:36:49"
tags: ["rsocket"]
categories: ["协议"]
---

RSocket 是一个新的通信协议， Facebook、Netfi、Pivotal、vlingo 等公司都有参与研发，它是一个位于传输层的新协议。在反应式编程中有两个瓶颈，一个是关系型数据库性能，一个是网络传输性能，对于数据库已经有[r2dbc](https://r2dbc.io/)开发，而网络问题则由 RScoket 解决，有人说 VaughnVernon 去 vlingo 做 RSocket 研究是为了解决领域驱动设计中上下文映射的问题（未求证）

## 为什么需要一个新协议

按[官方的解释](http://rsocket.io/docs/FAQ)翻译以及个人理解：

- 为了支持除了目前最常用的 Request/Respones 以外，像流式响应（类似于 Flink 的流处理）以及推送
- 二进制协议，使用字节流传输，支持单连接的多路复用
- 会话重连，这在移动端上很有用
- 需要有一个应用层的协议来支持自由切换像 WebSockets 和 Aeron 这样的传输层协议

## 支持的四种通讯模型

| 名称             | 用途                               |
| ---------------- | ---------------------------------- |
| Request/Response | rpc                                |
| Request/Stream   | pub/sub                            |
| Fire-and-Forget  | logging, metrics                   |
| channel          | mobile & IoT <-> Server 全双工通道 |



## Reactive 

RSocket 实现了 Reactive 编程模型，它有以下特点

- 全异步无阻塞
- 背压
- 并发模型 

## 参考文章

https://www.infoq.cn/article/2018%2F10%2Frsocket-facebook

https://www.ibm.com/developerworks/cn/java/j-using-rsocket-for-reactive-data-transfer/index.html

https://en.wikipedia.org/wiki/RSocket

https://dev.to/petros0/getting-started-with-rsocket-in-springboot-5889

http://rsocket.io/docs/FAQ











## 










