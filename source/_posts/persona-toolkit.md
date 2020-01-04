---
title: persona-toolkit
date: 2020-01-04 13:42:59
tags:
---

### pt-table-sync

- 使用对两个库不一致的数据进行同步，他能够自动发现两个实例间不一致的数据，然后进行sync操作，
- pt-table-sync无法同步表结构，和索引等对象，只能同步数据。
- 可能大家多数使用该工具来解决主从数据不一致的问题，其实他的功能不止于此，也可以用来对两个不在一个主从拓扑实例，进行数据sync。

NOTE1:
如果是sync主从数据，只有当需要sync的表都有唯一键(主键或唯一索引)，才能使用--sync-to-master and/or --replicate。
(没有唯一键，则只能在desitination上直接修改，而指定--sync-to-master and/or –replicate时只能在主库上修改)，
如果sync主从时没有指定--replicate或者--sync-to-master则所有修改都在从库上执行(不论表上是否有唯一键)


#### 使用示例
#### sync两个独立数据库(非主从)

以75为source，76为desitination，

1)同步所有的库和表
```
pt-table-sync --charset=utf8 --ignore-databases=mysql,sys,percona u=root,p=root,h=172.172.178.75,P=3306 u=root,p=root,h=172.172.178.76,P=3306 --execute --print
```
 
2) 同步指定库或者指定表

只对指定的库进行sync

```
pt-table-sync --charset=utf8 --ignore-databases=mysql,sys,percona --databases=test1 --no-check-slave u=root,p=root,h=172.172.178.75,P=3306 u=root,p=root,h=172.172.178.76,P=3306 --execute --print
```


只对指定的表进行sync(也可以--tables=test5.test_nu)

```
pt-table-sync --charset=utf8--ignore-databases=mysql,sys,percona --databases=test5 --tables=test_nu --no-check-slave dsn=u=root,p=root,h=172.172.178.75,P=3306 dsn=u=root,p=root,h=172.172.178.76,P=3306 --execute --print
```


 

1)忽略某些库或者忽略某些表

忽略库

--ignore-databases=指定要忽略的库

忽略表

pt-table-sync --charset=utf8--ignore-databases=mysql,sys,percona --ignore-tables=test5.test_nu --no-check-slave dsn=u=root,p=root,h=172.172.178.75,P=3306 dsn=u=root,p=root,h=172.172.178.76,P=3306 --execute --print
 

#### sync主从数据

1)没有唯一键(主键或唯一索引)


pt-table-sync --charset=utf8--ignore-databases=mysql,sys,percona --no-check-slave dsn=u=root,p=root,h=172.172.178.75,P=3306 dsn=u=root,p=root,h=172.172.178.76,P=3306 --execute --print
 


##只比对76同75的数据，对差异数据进行sync

可以指定多个从库


pt-table-sync --charset=utf8--ignore-databases=mysql,sys,percona --no-check-slave dsn=u=root,p=root,h=172.172.178.75,P=3306 dsn=u=root,p=root,h=172.172.178.76,P=3306 dsn=u=root,p=root,h=172.172.178.77,P=3306 --execute –print
 


##比对76同75；77同75的数据，对差异进行sync

 

NOTE1:在执行sync之前，我们可以指定—print(不要指定--execute)来查看pt-table-sync会进行哪些修改

NOTE2:执行sync操作时,指定忽略mysql等系统数据库(information_schema,performance_schema会自动被忽略)

NOTE3:指定--no-check-slave不检查desitination是否为从库，否则报如下错误:

Can't make changes onA=utf8,P=3306,h=172.172.178.76,p=... because it's a slave. See thedocumentation section 'REPLICATION SAFETY'

NOTE4:此种情况下修改都在从库上进行(不论表是否有唯一键)

 

2) 有唯一键(有唯一键时，可以使用--sync-to-master and/or –replicate进行主从sync)

pt-table-sync --execute --sync-to-master--charset=utf8 --ignore-databases=mysql,sys,percona --no-check-slave dsn=u=root,p=root,h=172.172.178.76,P=3306 --print
 

NOTE1:在执行sync之前，我们可以指定--print来查看pt-table-sync会进行哪些修改

NOTE2:执行sync操作时,指定忽略mysql等系统数据库(information_schema,performance_schema会自动被忽略)

NOTE3:如果表上没有唯一索引,则无法在主库执行replace操作，会报如下错此种情况不能指定--replicate或者--sync-to-master

 Can'tmake changes on the master because no unique index exists at/usr/bin/pt-table-sync line 10663.  whiledoing test1.test_concat on 172.172.178.76

NOTE4:指定了--replicate(指定dsn为主)或者--sync-to-master(指定dsn为从)只能为命令行指定一个dsn

参考
https://www.percona.com/doc/percona-toolkit/LATEST/pt-table-sync.html
https://www.hellojava.com/a/75316.html