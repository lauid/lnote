---
title: codis主从同步之后显示不一致问题
date: 2018-03-30 11:56:50
tags: codis
category: 运维
---

搭建codis2升级到codis3之后，
codis-server-2.x做master，codis-server-3.x做slave；
同步完成后发现主从的内存大小和keys数量完全不一致……

<!--more-->

![](/images/codis-image.png)


### salve日志

```
18006:S 29 Mar 13:34:11.294 # CONFIG REWRITE executed with success.
18006:S 29 Mar 13:34:11.884 * Connecting to MASTER 10.0.0.63:4379
18006:S 29 Mar 13:34:11.884 * MASTER <-> SLAVE sync started
18006:S 29 Mar 13:34:11.885 * Non blocking connect for SYNC fired the event.
18006:S 29 Mar 13:34:11.886 * Master replied to PING, replication can continue...
18006:S 29 Mar 13:34:11.886 * (Non critical) Master does not understand REPLCONF capa: -ERR Unrecognized REPLCONF option: capa
18006:S 29 Mar 13:34:11.886 * Partial resynchronization not possible (no cached master)
18006:S 29 Mar 13:34:11.886 * Full resync from master: 07703b01bfc5721ea125b3a483e4fe6c34266c7b:1
18006:S 29 Mar 13:34:37.704 * MASTER <-> SLAVE sync: receiving 657248822 bytes from master
18006:S 29 Mar 13:34:41.533 * MASTER <-> SLAVE sync: Flushing old data
18006:S 29 Mar 13:34:41.533 * MASTER <-> SLAVE sync: Loading DB in memory
18006:S 29 Mar 13:35:05.369 * MASTER <-> SLAVE sync: Finished with success
```
看着没什么异常.


### master的replication

```
role:master
connected_slaves:1
slave0:ip=10.0.0.64,port=14359,state=online,offset=224734716,lag=0
master_repl_offset:224734716
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:223686141
repl_backlog_histlen:1048576
```
master和slave的offset是一致的，所以说明主从的同步是一致完全的。


### 结论
那就是我们对下面几个参数的理解有问题了:
```
db0:keys=4155148,expires=4097878,avg_ttl=0	
```
表示0号数据库有 keys 个键、已经被删除的过期键数为expires、键到期的平均剩余时间为avg_ttl ms.

```
3179850-3135922 = 43,928
3179598-3135670 = 43,928

4156449-4099179 = 57,270
4155509-4098239 = 57,270
```
keys-expires主从相等说明是没问题的。



而salve的avg_ttl，redis是不统计的，所以一直是0

### 参考
- https://github.com/CodisLabs/codis/issues/645
- https://github.com/CodisLabs/redis-port/issues/35
- redis设计与实现

