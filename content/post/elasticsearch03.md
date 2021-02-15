---
title: "ElasticSearch API 基本操作"
date: "2019-05-27 17:52:23"
tags: ["ElasticSearch"]
categories: ["搜索"]
description: "ElasticSearch 7 的基本操作"
---

# 查看集群健康值

```
GET _cat/health?v
```

# 查看 node 信息
```
GET _nodes
```



# 查看索引信息

```
GET _cat/indices?v
```



# 创建索引

```
PUT /test?pretty
```

# 删除索引

```
DELETE /test?pretty
```



# 新建文档并建立索引
## 创建一个index为 ecommerce 的索引

```
PUT /ecommerce/_doc/1
{
  "product_id":1234,
  "product_name": "南极人",
  "price":99.9,
  "color":"Red",
  "tags":["轻松","舒服"]
}

PUT /ecommerce/_doc/2
{
 "product_id":5678,
  "product_name": "寓美",
  "price":199.9,
  "color":"White",
  "tags":["透气","实惠"]
}
PUT /ecommerce/_doc/3
{
  "product_id":23333,
  "product_name": "金利来",
  "price":1899.9,
  "color":"Blue",
  "tags":["奢侈","豪华"]
}
PUT /ecommerce/_doc/4
{
  "product_id":4444,
  "product_name": "南极人上衣",
  "price":19.9,
  "color":"Yellow",
  "tags":["保暖","豪华"]
}
```

# 查询操作
```
GET /ecommerce/_doc/1
```



# 更新操作方式1

```
PUT /ecommerce/_doc/1
{
  "product_id":1234,
  "product_name":"南极人裤子",
  "price":99.9,
  "color":"Red",
  "tags":["保暖","舒服"]
}
```



# 更新操作方式2
```
POST /ecommerce/_update/1
{
  "doc": {
    "product_id": 1234
  }

}
```



# 删除操作
```
DELETE /ecommerce/_doc/1
```




# DSL 查询
```
GET _search
{
  "query": {
    "match_all": {}
  },
  "timeout": "1ms"
}
```



# 查询并按价格排序
```
GET /ecommerce/_search
{
  "query": {
    "match": {
      "product_name": "南极人"
    }
  },
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}
```



# 指定返回的 field
```
GET /ecommerce/_search
{
  "query": {
    "match_all": {}
  },
  "_source":["product_id","product_name"]
}
```



# 分页查询 从 0 开始，每页一个
```
GET /ecommerce/_search
{
  "query": {"match_all": {}},
  "from": 0,
  "size": 1
}
```



# query filter
```
# https://www.elastic.co/guide/en/elasticsearch/reference/7.0/query-filter-context.html
GET /ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "product_name": "南极人"
          }
        }
      ],
      "filter": {"range": {
        "price": {
          "gte": 20,
          "lte": 100
        }
      }}
    }
  }
}
```



# 全文检索
## 只要 field 中有就返回

```
GET /ecommerce/_search
{
  "query": {
    "match": {
      "product_name": "南极人"
    }
  }
}
```



# highlight 
```
GET /ecommerce/_search
{
  "query": {
    "match": {
      "product_name": "南极人"
    }
  },
  "highlight": {
    "fields": {
      "product_name":{}
    }
  }
}
```



# fuzziness 模糊查询
## 先用全文检索

```
GET /ecommerce/_search
{
  "query": {
    "match": {
      "product_name": "南极人上衣"
    }
  }
}
```



# 开启模糊查询
```
GET /ecommerce/_search
{
  "query": {
    "match": {
      "product_name": {
        "query": "南极人上衣",
        "operator": "and"
      }
      }

  }
}
```



# phrase search
## 完全匹配

```
GET /ecommerce/_search
{
 "query": {
   "match_phrase": {
     "product_name": "南极人上衣"
   }
 }
}
```



# 聚合查询
## 正排

```
Set fielddata=true on [tags] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
PUT /ecommerce/_mapping
{
  "properties": {
    "tags":{
      "type": "text",
      "fielddata": true
    }
  }
}
```





# aggs 
```
GET /ecommerce/_search
{
  "aggs": {
    "whatever": {
      "terms": {
        "field": "tags",
        "size": 10
      }
    }
  }
}
```



# 先分组再求平均值,之后按平均值排序
```
GET /ecommerce/_search
{
  "query": {
    "match": {
      "product_name": "南极人"
    }
  }, 
  "aggs": {
    "NAME": {
      "terms": {
        "field": "tags",
        "order": {
          "avg_price": "asc"
        }, 
        "size": 10
      },
    "aggs": {
      "avg_price": {
       "avg": {
         "field": "price"
       }
      }
    }
    }
  }
}
```



# 区间
```
GET /ecommerce/_search
{
  "aggs": {
    "price_range": {
      "range": {
        "field": "price",
        "ranges": [
          {"from": 0,
            "to": 50
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100
            , "to": 150
          },
          {"from": 150,
            "to": 200
          }
        ]
      },
        "aggs": {
    "tags_group": {
      "terms": {
        "field": "tags",
        "size": 10
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
    }
  }
}
```

