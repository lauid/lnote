---
title: elasticsearch计算词频
date: 2019-07-23 09:01:43
tags:
---

安装环境：
elasticsearch-7.0
analysis_ik

```json
GET /sale_comment/_search
{
    "size": 0,
    "aggs":{
        "content":{
            "terms":{
                "size":1000,
                "field":"content",
                "include":"[\u4E00-\u9FA5]{2,6}",
                "exclude":"的.*|很.*|不.*|了.*|也.*|買.*|好.*|錯.*"
            }
        }
    }
}
```

```json
 "aggregations": {
      "content": {
         "doc_count_error_upper_bound": 0,
         "sum_other_doc_count": 205350,
         "buckets": [
            {
               "key": "舒服",
               "doc_count": 6051
            },
            {
               "key": "非常",
               "doc_count": 5583
            },
            {
               "key": "效果",
               "doc_count": 5289
            },
            {
               "key": "真的",
               "doc_count": 3404
            },
            {
               "key": "到了",
               "doc_count": 3383
            }
         ]
      }
   }
```
