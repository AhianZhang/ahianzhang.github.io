---
title: "阿里云 OpenSearch 介绍"
date: 2022-05-27T14:14:59+08:00
tags: ["OpenSearch"]
categories: ["总结"]
---

阿里云 OpenSearch 是一款基于 Ha3 引擎研发的商用搜索产品。在使用上给我的个人感受可以说 OpenSearch 是一套企业级搜索的完整解决方案的落地。下面我们来看一下 OpenSearch 有哪些功能。

![image-20220527163814607](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2022-05-27-083821.png)

# 现行搜索逻辑

## 下拉提示

使用 OpenSearch 官方算法模型，针对标题(title)、分类(category lines)、品牌(brand) 属性抽取值并结合高频搜索词进行自动训练

## 搜索召回逻辑

- 索引名称：default
- 分析方式：中文-电商分析
- 字段：标题(title)、款号(sku)、品牌(brand)、末级品类(category)、全品类(category_lines)
- 词典：同义词、停顿词

## 搜索排序逻辑

- 基础排序：static_bm25 ，静态文本相关性，用于衡量query与文档的匹配度
- 业务排序：无，此处可以优化，如果简单表达式不能满足要求则需要升级实例并使用 cava 进行自定义排序

## 数据采集

 ###  针对各端调用搜索时使用

- biz：来源，wechatshop、miniprogram、pc、ios、android
- userId：用户
- fromReqId：下拉提示点击进入搜索的 request id

### 当用户点击时记录

- 搜索的 requestId
- 商品 id
- 用户 id

## 主数据同步

全量更新：每天凌晨 1 点钟进行同步，可手动触发

增量更新：针对主数据变更近实时更新
