---
title: su指定用户执行命令
date: 2017-08-31 11:09:42
tags: crontab
categories: 运维
---

## 命令行模式下
```
/etc/passwd中nologin改为bash
su user -s /bin/bash -c 'touch test.x'
```

## /etc/crontab下 
```
* * * * * user echo '12312' >> /tmp/test.x
```
