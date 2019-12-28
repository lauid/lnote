---
title: elasticsearch-install
date: 2019-12-28 17:04:20
tags: elasticsearch
---

#### 下载tar包
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.0-linux-x86_64.tar.gz
```

<!--more--> 

#### 增加elastic用户
```
adduser elastic
```
解压包到elastic用户下

#### supervisor增加配置

```
[supervisord]
logfile=/var/log/supervisor.log
logfile_maxbytes=50MB        ; max main logfile bytes b4 rotation; default 50MB
logfile_backups=10           ; # of main logfile backups; 0 means none, default 10
loglevel=info                ; log level; default info; others: debug,warn,trace
pidfile=/var/run/supervisor.pid
nodaemon=false               ; start in foreground if true; default false
minfds=65535                    ; min. avail startup file descriptors; default 1024
minprocs=4096                   ; min. avail process descriptors;default 200
```
minfds=65535                    ; min. avail startup file descriptors; default 1024
minprocs=4096                   ; min. avail process descriptors;default 200

#### elasticsearch插件安装

```
./elasticsearch-7.0.0/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.0.0/elasticsearch-analysis-ik-7.0.0.zip

./elasticsearch-7.0.0/bin/elasticsearch-plugin install analysis-icu
bin/plugin install elasticsearch/elasticsearch-analysis-icu/VERSION

./elasticsearch-7.0.0/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-stconvert/releases/download/v7.0.0/elasticsearch-analysis-stconvert-7.0.0.zip

```
安装之后重启生效

