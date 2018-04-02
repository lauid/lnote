---
title: codis3.2后台展示一些说明
date: 2018-04-02 10:00:54
tags: codis
---

Session 中的：
total: 从 Proxy 启动到当前时间已经处理的连接数
alive: 当前 Porxy 中正在处理的连接数
Commands 中的：
total: 从 Proxy 启动到当前时间已经处理的命令个数
fails: 从 Proxy 启动到当前时间处理失败的命令个数
rsp.errs: 从 Proxy 启动到当前时间 Redis 返回 Errror 的命令个数
