---
title: "RabbitMQ connection channel 的关系 "
date: 2022-09-07T10:27:45+08:00
tags: ["rabbitmq"]
categories: ["总结"]
---

Rabbit java client 发送消息代码如下

```java
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        AMQP.BasicProperties basicProperties = new AMQP.BasicProperties().builder()
          .messageId(UUID.randomUUID().toString())
          .build();
        channel.basicPublish("biz","dev.hello",false,basicProperties,"Hello World".getBytes());
        channel.close();
        connection.close();
```

![image-20220907115304492](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2022-09-07-035306.png)

对于 AMQP 0-9-1 网络连接，一个客户端会与 Broker 建立一个 Tcp 长连接（Connection），并使用轻量级连接（Channel）来实现多路复用的效果。
