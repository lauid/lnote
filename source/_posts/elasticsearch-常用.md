---
title: elasticsearch-常用
date: 2019-12-04 09:57:54
tags: elasticsearch
---


#### 分词测试

```json
GET /goods_index/_analyze
{
  "analyzer" : "icu_analyzer",
  "text" : "Elasticsearch hello world"
}
```

#### cmd只更新部分字段

```json
POST /goods_index/_doc/103130673/_update
{
    "doc":{
        "categoryIds": [1,2,3]
    }
}
```
