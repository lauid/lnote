---
uuid: 05dc7350-4acf-11e9-b789-9379655bdfa8
title: mysql的innodb幻读问题
date: 2019-03-20 13:14:22
tags:
---

[MySQL InnoDB事务的隔离级别](https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html)有四级，默认是“可重复读”（REPEATABLE READ）。

未提交读（READ UNCOMMITTED）。另一个事务修改了数据，但尚未提交，而本事务中的SELECT会读到这些未被提交的数据（脏读）。
提交读（READ COMMITTED）。本事务读取到的是最新的数据（其他事务提交后的）。问题是，在同一个事务里，前后两次相同的SELECT会读到不同的结果（不重复读）。
可重复读（REPEATABLE READ）。在同一个事务里，SELECT的结果是事务开始时时间点的状态，因此，同样的SELECT操作读到的结果会是一致的。但是，会有幻读现象（稍后解释）。
串行化（SERIALIZABLE）。读操作会隐式获取共享锁，可以保证不同事务间的互斥。
四个级别逐渐增强，每个级别解决一个问题。

脏读，最容易理解。另一个事务修改了数据，但尚未提交，而本事务中的SELECT会读到这些未被提交的数据。
不重复读。解决了脏读后，会遇到，同一个事务执行过程中，另外一个事务提交了新数据，因此本事务先后两次读到的数据结果会不一致。
幻读。解决了不重复读，保证了同一个事务里，查询的结果都是事务开始时的状态（一致性）。但是，如果另一个事务同时提交了新数据，本事务再更新时，就会“惊奇的”发现了这些新数据，貌似之前读到的数据是“鬼影”一样的幻觉。
借鉴并改造了一个搞笑的比喻：

脏读。假如，中午去食堂打饭吃，看到一个座位被同学小Q占上了，就认为这个座位被占去了，就转身去找其他的座位。不料，这个同学小Q起身走了。事实：该同学小Q只是临时坐了一小下，并未“提交”。
不重复读。假如，中午去食堂打饭吃，看到一个座位是空的，便屁颠屁颠的去打饭，回来后却发现这个座位却被同学小Q占去了。
幻读。假如，中午去食堂打饭吃，看到一个座位是空的，便屁颠屁颠的去打饭，回来后，发现这些座位都还是空的（重复读），窃喜。走到跟前刚准备坐下时，却惊现一个恐龙妹，严重影响食欲。仿佛之前看到的空座位是“幻影”一样。
------

一些文章写到InnoDB的可重复读避免了“幻读”（phantom read），这个说法并不准确。

- 获得当前会话的TRX_ID 事务ID
> start transaction; 
> SELECT TRX_ID FROM INFORMATION_SCHEMA.INNODB_TRX  WHERE TRX_MYSQL_THREAD_ID = CONNECTION_ID();
> commit;

- 查询 正在执行的事务：
> SELECT * FROM information_schema.INNODB_TRX

- 查看正在锁的事务
> SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;

- 查看等待锁的事务

> SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;


如果使用普通的读，会得到一致性的结果，如果使用了加锁的读，就会读到“最新的”“提交”读的结果。

本身，可重复读和提交读是矛盾的。在同一个事务里，如果保证了可重复读，就会看不到其他事务的提交，违背了提交读；如果保证了提交读，就会导致前后两次读到的结果不一致，违背了可重复读。

可以这么讲，InnoDB提供了这样的机制，在默认的可重复读的隔离级别里，可以使用加锁读去查询最新的数据。


> http://dev.mysql.com/doc/refman/5.0/en/innodb-consistent-read.html
> 
> If you want to see the “freshest” state of the database, you should use either the READ COMMITTED isolation level or a locking read:
> SELECT * FROM t_bitfly LOCK IN SHARE MODE;

------

结论：MySQL InnoDB的可重复读并不保证避免幻读，需要应用使用加锁读来保证。而这个加锁度使用到的机制就是next-key locks。







