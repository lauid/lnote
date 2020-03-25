---
title: mysql区分大小写问题
date: 2020-03-25 12:36:10
tags: mysql
---

mysql字符串区分大小写的问题

### 一、

##### 1、 CREATE TABLE NAME(name VARCHAR(10));

对这个表，缺省情况下，下面两个查询的结果是一样的：

SELECT * FROM TABLE NAME WHERE name='clip';

SELECT * FROM TABLE NAME WHERE name='Clip';

> MySql默认查询是不区分大小写的,如果需要区分他,必须在建表的时候,Binary标示敏感的属性.

CREATE TABLE NAME( name VARCHAR(10) BINARY);

#### 2、 在SQL语句中实现 SELECT * FROM TABLE NAME WHERE BINARY name='Clip';

#### 3、 设置字符集：

utf8_general_ci --不区分大小写

utf8_bin--区分大小写

二、 MySQL在windows下是不区分大小写的，将script文件导入MySQL后表名也会自动转化为小写，结果再 想要将数据库导出放到linux服务器中使用时就出错了。
因为在linux下表名区分大小写而找不到表，查了很多都是说在linux下更改MySQL的设置使其也不区分大小写，但是有没有办法反过来让windows 下大小写敏感呢。
其实方法是一样的，相应的更改windows中MySQL的设置就行了。

具体操作：

在MySQL的配置文件my.ini中增加一行：

lower_case_table_names = 0

其中 0：区分大小写，1：不区分大小写

MySQL在Linux下数据库名、表名、列名、别名大小写规则是这样的：

1、数据库名与表名是严格区分大小写的；

2、表的别名是严格区分大小写的；

3、列名与列的别名在所有的情况下均是忽略大小写的；

4、变量名也是严格区分大小写的； MySQL在Windows下都不区分大小写

http://www.jb51.net/article/61591.htm

当我们输入不管大小写都能查询到数据，例如：输入 aaa 或者aaA ,AAA都能查询同样的结果，说明查询条件对大小写不敏感。
解决方案一：
于是怀疑Mysql的问题。做个实验：直接使用客户端用sql查询数据库。 发现的确是大小不敏感 。
通过查询资料发现需要设置collate（校对） 。 collate规则：
*_bin: 表示的是binary case sensitive collation，也就是说是区分大小写的
*_cs: case sensitive collation，区分大小写
*_ci: case insensitive collation，不区分大小写

解决方法一：
1.可以将查询条件用binary()括起来。 比如： 
select * from TableA where binary columnA ='aaa';
2. 可以修改该字段的collation 为 binary
比如：
ALTER TABLE TABLENAME MODIFY COLUMN COLUMNNAME VARCHAR(50) BINARY CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL;

解决方案二：
mysql查询默认是不区分大小写的 如:
select * from some_table where str=‘abc';
select * from some_table where str='ABC';
得到的结果是一样的，如果我们需要进行区分的话可以按照如下方法来做： 
第一种方法：
要让mysql查询区分大小写，可以：
select * from some_table where binary str='abc'
select * from some_table where binary str='ABC'
第二种方法：
在建表时时候加以标识
create table some_table(
str char(20) binary 
)
原理：
对于CHAR、VARCHAR和TEXT类型，BINARY属性可以为列分配该列字符集的 校对规则。
BINARY属性是指定列字符集的二元 校对规则的简写。
排序和比较基于数值字符值。因此也就自然区分了大小写。


