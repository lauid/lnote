---
title: SQL优化器soar
date: 2020-03-13 09:18:35
tags: sql
---

下载 XiaoMi 开源的 SQL 优化器 soar，更多详细安装请参考 soar install
```
wget https://github.com/XiaoMi/soar/releases/download/0.11.0/soar.linux-amd64
chmod a+x soar 
​echo “select * from tb_area” | ./soar -test-dsn="数据库用户名:数据库密码@127.0.0.1:3306/my_base"
```
输出的信息拷贝到markdown工具下即可直观查阅提示信息 或者把结果输出到文件中

content=$(cat query.sql)
echo $content | ./soar.linux-amd64 -test-dsn="root:123456@127.0.0.1:3306/intellif_base" > file.txt



参考:

https://github.com/guanguans/soar-php