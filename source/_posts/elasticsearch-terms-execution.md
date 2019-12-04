---
title: elasticsearch-terms-execution
date: 2019-12-03 22:33:55
tags:
---

```json
{
    "terms" : { "field_name" : ["value1", "value2"], "execution" : "bool" }
}

```
> "reason": "[terms] query does not support [execution]",
这种方式已经不支持了，目前只有用must没有找到更优雅的写法。

```json
POST /goods_index/_search
{
   "explain": true,
   "query": {
      "bool": {
         "filter": [
            {
               "bool": {
                  "must": [
                     {
                        "term": {
                           "categoryIds": {
                              "value": 1
                           }
                        }
                     },
                     {
                        "term": {
                           "categoryIds": {
                              "value": 2
                           }
                        }
                     }
                  ]
               }
            }
         ]
      }
   }
}
```

ps. 
- https://www.elastic.co/guide/cn/elasticsearch/guide/current/_finding_multiple_exact_values.html
- https://discuss.elastic.co/t/is-there-an-alternative-solution-to-terms-execution-and-on-es-2-x/41089
- https://github.com/elastic/elasticsearch/issues/1568
