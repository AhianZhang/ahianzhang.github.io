---
title: "Kafka Java 客户端 Producer 原理分析"
date: 2022-06-06T13:41:05+08:00
tags: ["Kafka"]
categories: ["总结"]
draft: true
---

Kafka Producer Java 客户端如下：

```java
public class MyProducer {
    static String topic = "test";
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        /*
         0 生产者发送消息之后不需要等待任何服务端的响应。
         1 生产者发送消息之后，只要分区的 leader 副本成功写入消息，那么它就会收到来自服务端的成功响应，但是如果发生在 leader 选举阶段会造成消息丢失
        -1/all 生产者在消息发送之后，需要等待 ISR 中的所有副本都成功写入消息之后才能够收到来自服务端的成功响应。
         */
        props.put("acks", "all");
        // 重试次数
        props.put("retries", 0);
        // 指定 Buffer Pool 大小，超过
        props.put("batch.size", 16384);
        // ProducerBatch 发送时间，如果 batch.size 未达到，但是 linger ms 时间达到，则会发送次 Batch 消息
        props.put("linger.ms", 10000); // 10s 发送
        //RecordAccumulator 缓存大小，默认值为 33554432B，32MB
        props.put("buffer.memory", 33554432);
        // 生成消息的速度大于发送的速度会造成生产者内存不足，要么抛异常，要么阻塞这个配置的毫秒数
        props.put("max.block.ms",60000);
        // 消息最大容量，默认 1MB
        props.put("max.request.size",1048576);

        props.put("key.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");

        props.put("value.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");

        Producer<String,String> producer = new KafkaProducer<>(props);

        for (int i = 0; i < 10; i++) {
            Thread.sleep(1000);
            producer.send(new ProducerRecord<>(topic,
                    Integer.toString(i), Integer.toString(i)), (recordMetadata, exception) -> {
                System.out.printf("topic=%s,partition=%s,offset=%d,timestamp=%s \n",
                        recordMetadata.topic(),recordMetadata.partition(),
                        recordMetadata.offset(),recordMetadata.timestamp());
                    });
        }
        System.out.println("Message sent successfully");
        producer.close();
    }

```

从代码结构上分析，主要是两个部分：1. Producer 参数配置； 2. 发送消息。

Producer 参数部分是控制 Kafka 高效稳定发送消息的关键，常用的参数已经写到代码中。如果需要查看全部 Producer 参数，可以在代码开启 Debug 模式运行查看。

我们主要分析一下 Send 的原理

