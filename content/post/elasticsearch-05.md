---
title: "Elasticsearch 检索性能优化"
date: 2024-12-20T15:07:28+08:00
tags: ["ElasticSearch"]
categories: ["搜索","性能优化"]
---

# 前言

ElasticSearch 是当前使用场景比较多的全文检索数据库，可以用到搜广推业务、日志监控、数据可视化等。既然是数据库就无法避免的会遇到查询性能问题，本篇文章主要介绍从索引设计、集群配置、查询三个方面进行优化。

## 索引优化

### 设置 ``index:false``

适用场景：对没有检索和过滤需求的并且后续不会更新的字段，这个设置只是不建立索引，实际数据还是会进行存储，在查询时可以通过 `_source` 进行返回。

### 设置 shard 数

基础概念可以参考[之前的文章](https://www.ahianzhang.com/post/elastic-search/)

简单讲一下 shard 的开销，shard 分为主分片和副本分片两种，那如果 shard 数越多，需要的同步时间就会越久，检索时会发生数据短暂不一致情况，并且同步的成本及稳定性也会受影响；shard 越多需要分配的内存和 CPU 也就越多，如果并发量大的话会将 node 的线程池耗尽，从而降低吞吐量。

官方推荐，理论上一个 shard 可以存储 **20 亿文档**，或者 **10G-50G** 之间的数据，如果你的数据在这两个范围内，可以直接使用一个 shard 进行检索。

### 设置副本数

这个副本数是和主 shard 一一对应的，也就是现在有三个 primary shard，如果副本数设置为 1，那么会出现三个 replica shard，确保每个主分片都有对应的副本，如果副本数不够分配，集群状态就会变成黄色。在生产环境至少要配置 1 个分片，在本地开发可以设置为 0。

举例：如果数据预估是 1T，shard 分片大小为 50 G，那就需要 20 个 primary shard，如果副本数设置为 1，那么也会存在 20 个 replica shard，一共是 40 个 shard。

### 禁用 Dynamic Mapping

使用确定的映射，不要让 ES 自己推断，这个在日常设计中都会注意。

### 慎用复杂结构

官方指出，nested 结构查询会让搜索慢几倍，而 parent-child 会让搜索慢上百倍！

如果要存储简单的键值对，并且后期只会用到简单的查询聚合场景，可以将字段设置为`` flattened ``类型，它不会单独维护一个子文档

### 使用 ``copy_to``

把经常要组合搜索的放到同一个字段中，比如要同时在文章标题、正文中查找，那可以做如下设置：

```
{
  "mappings": {
    "properties": {
      "full-text": { // 只需要用这个字段进行搜索
        "type": "text"
      },
      "title": {
        "type": "text",
        "copy_to": "name_and_plot"
      },
      "content": {
        "type": "text",
        "copy_to": "name_and_plot"
      }
    }
  }
}
```

### 数据预处理

比如日期，2024-12-22 10:00:00，我们可以拆成多个字段进行存储

```
{
  "mappings": {
    "properties": {
      "published": { 
        "type": "date"
      },
      "year": { 
        "type": "keyword"
      },
      "month": {
        "type": "keyword",
        
      },
      "day": {
        "type": "keyword",
      }
    }
  }
}
```

### 优先使用 Keyword

对于不需要范围查询的，比如身份证号、ISBN 这种，要使用 Keyword 类型存储，因为 keyword 查询要比数值类型的速度快。

### 段合并

此操作对经常变更的索引没有太大帮助，但是如果已经不会再变的数据，比如昨日索引的数据，使用段合并能提高查询效率。

## 集群配置

### 合理设置 node 的角色

ES 中的 node 分为 

- Master Node：负责管理集群的元数据和控制集群的状态，例如索引创建、分片分配等（保证集群高可用和稳定性，不存储数据）
- Data Node：存储实际的数据和实际的查询请求（存储和计算）
- Coordinating：协调节点，负责搜索和聚合的请求分发和结果合并，不存储数据，类似于微服务中的服务编排。
- Ingest：数据预处理节点，如果数据处理操作在代码中完成，这个可以不设计。

1. 在生产和测试环境要显式设置集群角色，不要混用，配置项：``node.roles``
2. 对于 Master 角色，设置奇数个，避免脑裂问题

### 合理分配 node 的资源

#### Master 节点

master 只负责集群的管理，所以不需要太多的资源

```
# JVM 堆内存配置
-Xms4g
-Xmx4g

# 系统配置
CPU: 4-8 核
内存: 8GB
磁盘: 普通 SATA 即可，100GB
```

原因：

- 主要处理集群状态管理，不需要太大内存
- 集群元数据相对较小
- 需要稳定的 CPU 处理能力
- 磁盘 I/O 要求不高

#### Data 节点

数据节点要重点关照

```
# JVM 堆内存配置
-Xms31g
-Xmx31g  # 保持在32GB以下，避免指针压缩失效

# 系统配置
*CPU: 8-32 核
*内存: 64GB+
**磁盘: SSD，500GB+
```

原因：

- 需要大内存做缓存
- 需要留足够系统内存给 Lucene 文件系统缓存
- 多核 CPU 有利于并行处理
- SSD 提高随机 I/O 性能

指针压缩会降低存储成本，能够让  CPU 加载更多的对象引用，提高 CPU 缓存命中率。最新的 JDK 24 版本中对此做了优化 [文章地址]([https://www.infoq.com/news/2024/11/compact-headers-java24/)

堆内存的大小不要超过物理内存的 50%，剩下空间的主要给 Lucene 用来做缓存。也就是说 JVM 配置了 31 G，那么物理内存至少是 64 G。

#### Coordinating 节点

```
# JVM 堆内存配置
-Xms16g
-Xmx16g

# 系统配置
*CPU: 8-16 核
*内存: 32GB
磁盘: 普通 SATA，200GB
```

原因：

- 需要足够内存做查询缓存
- 需要较强 CPU 处理聚合计算
- 磁盘要求不高

## 查询优化

### 显式设置返回字段

对应业务无关的数据可以在查询时使用`` _source`` 进行设置

```
"_source": {
  "includes": ["field1", "field2"]
}
```

### 使用 filter 查询

对于复杂的查询条件并且不需要排序的场景，要使用 filter+bool 的方式

```
{
  "query": {
    "bool": {
      "filter": [
        {"term": {"userLevel": "PGC"}}
      ]
    }
  }
}
```

### 不要一次性返回大集合

如果需要数据导出可以使用 Scroll 或者 SearchAfter，每次条数可以根据实际的文档大小进行设置，要权衡吞吐量和服务稳定性。

### 高亮后置处理

在程序代码中进行筛选和飘红处理，简单讲就是将查询 keyword 分词，然后遍历返回内容，如果匹配到就飘红处理。

### 慎用 Script

脚本查询虽然使用灵活，但是因为需要动态计算，会降低整体的性能

### 聚合显式设置 collect_mode

在使用聚合统计时，有一个``collect_mode`` 参数可以设置，它有两种值：

- BFS：广度优先，第一步先将顶层 bucket 计算完，再计算里面的子集，在 bucket 数量比较小的时候使用，如统计男女数量、正负面等。
- DFS：深度优先，计算出一个顶层 bucket后计算这个 bucket 的子集，在 bucket 数量很大的时候使用，如统计价格分布数量、互动量等。

默认情况下，ES 会自动推断，但是如果明确聚合 bucket 数量大小时，建议手动设置。

# 总结

ES 性能优化是一个非常复杂的事情，需要从物理字段、JVM、索引、查询等方面进行综合考虑，上面给出只是优化的一些手段和建议，在实战时还需要具体问题具体分析。在做方案设计时早期要考虑的详细一些，确定集群的规格参数，因为后期物理层面的成本会很大。

# 参考文章

https://www.elastic.co/guide/en/elasticsearch/reference/8.17/size-your-shards.html#each-shard-has-overhead

https://www.elastic.co/guide/en/elasticsearch/reference/8.17/tune-for-search-speed.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/flattened.html

