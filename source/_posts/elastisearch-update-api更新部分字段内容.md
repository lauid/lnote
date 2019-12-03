---
title: elastisearch-update-api更新部分字段内容
date: 2019-12-03 22:51:04
tags:
---

https://www.elastic.co/guide/cn/elasticsearch/guide/current/partial-updates.html
update 请求最简单的一种形式是接收文档的一部分作为 doc 的参数， 它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段。

```json
POST /website/blog/1/
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}
```

例如，我们增加字段 tags 和 views 到我们的博客文章，如下所示：

```json
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```
