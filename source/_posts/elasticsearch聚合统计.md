---
title: elasticsearch聚合统计
date: 2020-01-03 10:08:02
tags: elasticsearch
---

Elasticsearch的Aggregation功能也异常强悍。

Aggregation共分为三种：Metric Aggregations、Bucket Aggregations、 Pipeline Aggregations。下面将分别进行总结。

 
<!--more-->

以下所有内容都来自官网：

官网（权威）：https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-avg-aggregation.html

 

#########################################

#### 1、Metric Aggregations

##### 1>[Avg Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-avg-aggregation.html)  #计算出字段平均值

{
    "aggs" : {
        "avg_grade" : { "avg" : { "field" : "grade" } }
    }
}
例子：

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "avg_grade": {
      "avg": {
        "field": "grade"
      }
    }
  }
}

参数：search_type=count 表示只返回aggregation部分的结果。


##### 2>[Cardinality Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-cardinality-aggregation.html)  #计算出字段的唯一值。相当于sql中的distinct

> cardinality 度量是一个近似算法。 它是基于 HyperLogLog++ （HLL）算法的。 HLL 会先对我们的输入作哈希运算，然后根据哈希运算的结果中的 bits 做概率估算从而得到基数。
> 优点：性能快，亿级别的记录在1秒内完成
> 缺点：存在只能保证最大40000条记录内的精确，超过的存在5%的误差，不适合需要精确去重场景

{
    "aggs" : {
        "author_count" : {
            "cardinality" : {
                "field" : "author"
            }
        }
    }
}
例子：

 

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "author_count": {
      "cardinality": {
        "field": "author"
      }
    }
  }
}

 

##### 3>[Extended Stats Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-extendedstats-aggregation.html) #字段的其他属性，包括最大最小，方差等等。

 

{
    "aggs" : {
        "grades_stats" : { "extended_stats" : { "field" : "grade" } }
    }
}
 

 

例子：GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "grades_stats": {
      "extended_stats": {
        "field": "grade"
      }
    }
  }
}

返回值：

 

{
    ...

    "aggregations": {
        "grade_stats": {
           "count": 9,
           "min": 72,
           "max": 99,
           "avg": 86,
           "sum": 774,
           "sum_of_squares": 67028,
           "variance": 51.55555555555556,
           "std_deviation": 7.180219742846005,
           "std_deviation_bounds": {
            "upper": 100.36043948569201,
            "lower": 71.63956051430799
           }
        }
    }
}
 

 

##### 4>[Geo Bounds Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-geobounds-aggregation.html) #计算出所有的地理坐标将会落在一个矩形区域。比如说朝阳区域有很多饭店，我就可以用一个矩形把这些饭店都圈起来，看看范围。

{
    "query" : {
        "match" : { "business_type" : "shop" }
    },
    "aggs" : {
        "viewport" : {
            "geo_bounds" : {
                "field" : "location", 
                "wrap_longitude" : true 
            }
        }
    }
}


例子：

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "viewport": {
      "geo_bounds": {
        "field": "location",
        "wrap_longitude": true
      }
    }
  }
}

返回值：

 

{
    ...

    "aggregations": {
        "viewport": {
            "bounds": {
                "top_left": {
                    "lat": 80.45,
                    "lon": -160.22
                },
                "bottom_right": {
                    "lat": 40.65,
                    "lon": 42.57
                }
            }
        }
    }
}
 

 

注释：这个矩形区域左上角坐标，和右下角坐标已经给出。也就是说你查出来的数据将会都落在这个地理范围内。

 

 

##### 5>[Geo Centroid Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-geocentroid-aggregation.html)   #计算出所有文档的大概的中心点。比如说某个地区盗窃犯罪很多，那我这样就可以看到这片区域到底哪个点（街道）偷盗事件最猖狂。

 

{
    "query" : {
        "match" : { "crime" : "burglary" }
    },
    "aggs" : {
        "centroid" : {
            "geo_centroid" : {
                "field" : "location" 
            }
        }
    }
}
 

 

例子：

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "centroid": {
      "geo_centroid": {
        "field": "location"
      }
    }
  }
}

 

##### 6>[Max Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-max-aggregation.html)  #求最大值

 

{
    "aggs" : {
        "max_price" : { "max" : { "field" : "price" } }
    }
}
例子：

 

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "max_price": {
      "max": {
        "field": "price"
      }
    }
  }
}

 

##### 7>[Min Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-min-aggregation.html) #求最小值

 

{
    "aggs" : {
        "min_price" : { "min" : { "field" : "price" } }
    }
}
例子：

 

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "min_price": {
      "min": {
        "field": "price"
      }
    }
  }
}

 

 

##### 8>[Percentiles Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-percentile-aggregation.html) #百分比统计。可以看出你网站的所有页面。加载时间的差异。

 

{
    "aggs" : {
        "load_time_outlier" : {
            "percentiles" : {
                "field" : "load_time" 
            }
        }
    }
}
例子：

 

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "load_time_outlier": {
      "percentiles": {
        "field": "load_time"
      }
    }
  }
}

 

返回：可以看出这个网站75%页面在29毫秒左右就加载完毕了。有5%的页面超过了60毫秒。

 

{
    ...

   "aggregations": {
      "load_time_outlier": {
         "values" : {
            "1.0": 15,
            "5.0": 20,
            "25.0": 23,
            "50.0": 25,
            "75.0": 29,
            "95.0": 60,
            "99.0": 150
         }
      }
   }
}
 

##### 9>[Percentile Ranks Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-percentile-rank-aggregation.html) #看看15毫秒和30毫秒内大概有多少页面加载完。

 

{
    "aggs" : {
        "load_time_outlier" : {
            "percentile_ranks" : {
                "field" : "load_time", 
                "values" : [15, 30]
            }
        }
    }
}
 

例子：

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "load_time_outlier": {
      "percentile_ranks": {
        "field": "load_time",
        "values": [
          15,
          30
        ]
      }
    }
  }
}

 

返回：看出15毫秒时大概92%页面加载完毕。30毫秒时基本都加载完成。

 

{
    ...

   "aggregations": {
      "load_time_outlier": {
         "values" : {
            "15": 92,
            "30": 100
         }
      }
   }
}
 

 

##### 10>[Stats Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-stats-aggregation.html)  #最大、最小、和、平均值。一起求出来

 

{
    "aggs" : {
        "grades_stats" : { "stats" : { "field" : "grade" } }
    }
}
 

例子：

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "grades_stats": {
      "stats": {
        "field": "grade"
      }
    }
  }
}

 

##### 11>[Sum Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-sum-aggregation.html) #求和

 

 "aggs" : {
        "intraday_return" : { "sum" : { "field" : "change" } }
    }
例子：

 

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "intraday_return": {
      "sum": {
        "field": "change"
      }
    }
  }
}

 

##### 12>[Top hits Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-top-hits-aggregation.html)  #较为常用的统计。获取到每组前n条数据。相当于sql 中 group by 后取出前n条。 

 

{
    "aggs": {
        "top-tags": {
            "terms": {
                "field": "tags",
                "size": 3
            },
            "aggs": {
                "top_tag_hits": {
                    "top_hits": {
                        "sort": [
                            {
                                "last_activity_date": {
                                    "order": "desc"
                                }
                            }
                        ],
                        "_source": {
                            "include": [
                                "title"
                            ]
                        },
                        "size" : 1
                    }
                }
            }
        }
    }
}
例子：取100组，每组只要第一条。为了见bain没用order和_source，请自行测试他们。

 

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "zxw_id",
        "size": 100
      },
      "aggs": {
        "top_tag_hits": {
          "top_hits": {
            "size": 1
          }
        }
      }
    }
  }
}

 

##### 14>[Value Count Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-metrics-valuecount-aggregation.html)  #数量统计，看看这个字段一共有多少个不一样的数值。

{
    "aggs" : {
        "grades_count" : { "value_count" : { "field" : "grade" } }
    }
}
例子：

 

GET index/type/_search?search_type=count
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "grades_count": {
      "value_count": {
        "field": "grade"
      }
    }
  }
}

 

#### 2、[Bucket Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-bucket.html) 这是第二种类型的统计（用的也是最多的，最实用的。）

网站：https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-bucket-children-aggregation.html

 

#### 3、[Pipeline Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations-bucket.html) #这是第三中类型的聚合。

