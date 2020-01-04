---
title: jira破解安装
date: 2020-01-04 16:52:28
tags:
---

1、官网下载

www.atlassian.com/software/ji…
https://product-downloads.atlassian.com/software/jira/downloads/atlassian-jira-software-8.6.0.tar.gz

2、下载Mysql JDBC驱动,文件解压到jira解压文件lib中。

https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.48.tar.gz

3、复制破解文件，替换原文件。

/atlassian-jira-software-8.0.2-standalone/WEB-INF/lib`

4、解压后,执行config.sh脚本（直接拖拽到终端），设置Jira_home、关联数据库

路径：/atlassian-jira-software-8.0.2-standalone/bin

5、启动Jira，执行start-jira.sh脚本

/atlassian-jira-software-8.0.2-standalone/bin

6、配置jira，http://localhost:8080/ ，配置过程会选择内置或者关联数据库，内置适合本机试用可以不安装数据库，服务器部署选择另一个。根据id生成一个License(点击生成JIRA试用许可证)，登陆配置完成。