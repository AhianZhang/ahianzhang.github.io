---
title: "elasticsearch 集群容错"
date: "2019-05-24 16:15:45"
tags: ["ElasticSearch"]
categories: ["搜索"]
---



**Q:有三台服务器，三个 Primary Shard怎么配置能使 ElasticSearch 达到高可用**



在此之前应该先清楚 shard 、primary shard 、replica shard、node 的概念

还有 primary shard 在确定好个数后后期是无法更改的，能扩容的只有 replica shard。

三个 Primary Shard 的意思就是：

```
3 primary + 3 replica = 6 shard (默认情况下)
```

此时服务器的 node 分配如下：

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/Paper.Paper%20%E5%B7%A5%E5%85%B7.18.png)

## 一台服务器宕机

此时整个 ES 集群是能够正常提供服务的，但是健康值会降到 yellow 状态

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/IMG_0038.PNG)

## 两台服务器宕机

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/IMG_0039.PNG)

此时你会发现剩下的一台服务器上只会有一个 primary shard 和 一个 replica shard 节点，此时集群状态是 red 转态。

怎么去分配能够使两台机器宕机后仍能正常使用？

## 设置更多的 replica shard

既然 primary shard 初始化后不能再扩容了，那么可以扩容 replica shard。

如下图，每台机器上不论是 primary 还是 replica shard 都保持了均与分布的状态，此时两台机器宕机后 ES 集群会自动将剩余的节点全都置位 primary shard。其内部原理将会在后面讲到。

![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/IMG_0040.PNG)

# 总结

虽然 replica shard 可以在后期调整，但是最佳的做法就是在确认架构的时候，而不是发现问题后再进行补救。