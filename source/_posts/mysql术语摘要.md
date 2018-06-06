---
title: mysql术语摘要
date: 2018-05-04 11:05:01
tags: 翻译
---

> 原文：https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_crash_recovery

<!--more-->

## ACID

An acronym standing for atomicity, consistency, isolation, and durability. These properties are all desirable in a database system, and are all closely tied to the notion of a transaction. The transactional features of InnoDB adhere to the ACID principles.

Transactions are atomic units of work that can be committed or rolled back. When a transaction makes multiple changes to the database, either all the changes succeed when the transaction is committed, or all the changes are undone when the transaction is rolled back.

The database remains in a consistent state at all times — after each commit or rollback, and while transactions are in progress. If related data is being updated across multiple tables, queries see either all old values or all new values, not a mix of old and new values.

Transactions are protected (isolated) from each other while they are in progress; they cannot interfere with each other or see each other's uncommitted data. This isolation is achieved through the locking mechanism. Experienced users can adjust the isolation level, trading off less protection in favor of increased performance and concurrency, when they can be sure that the transactions really do not interfere with each other.

The results of transactions are durable: once a commit operation succeeds, the changes made by that transaction are safe from power failures, system crashes, race conditions, or other potential dangers that many non-database applications are vulnerable to. Durability typically involves writing to disk storage, with a certain amount of redundancy to protect against power failures or software crashes during write operations. (In InnoDB, the doublewrite buffer assists with durability.)

See Also atomic, commit, concurrency, doublewrite buffer, isolation level, locking, rollback, transaction.


```
ACID 

缩写代表原子性，一致性，隔离性和耐久性。这些属性在数据库系统中都是可取的，并且都与事务的概念密切相关 。InnoDB坚持ACID原则的事务特征。

事务是可以提交或回滚的原子工作单元。当事务对数据库进行多重更改时，或者在事务提交时所有更改都成功，或者在事务回滚时撤消所有更改。

数据库始终保持一致状态 - 每次提交或回滚之后，以及事务处理正在进行中。如果相关数据在多个表中进行更新，则查询会查看所有旧值或全部新值，而不是旧值和新值的混合。

事务在进行时会相互保护（隔离）; 他们不能互相干扰或看到对方未提交的数据。这种隔离是通过锁定机制实现的。有经验的用户可以调整隔离级别，当他们可以确定事务真的不会互相干扰时，可以减少保护，以提高性能和并发性。

事务的结果是持久的：一旦提交操作成功，该事务所做的更改对电源故障，系统崩溃，竞态条件或许多非数据库应用程序易受攻击的其他潜在危险都是安全的。耐久性通常涉及到写入磁盘存储，并具有一定的冗余度以防止写操作期间的电源故障或软件崩溃。（在 InnoDB，双写缓冲区协助耐用性。）

另请参见atomic，commit，concurrency，doublewrite buffer，隔离级别，锁定，回滚，事务。

```

## atomic

In the SQL context, transactions are units of work that either succeed entirely (when committed) or have no effect at all (when rolled back). The indivisible ("atomic") property of transactions is the “A” in the acronym ACID.

See Also ACID, commit, rollback, transaction.

```
原子

在SQL上下文中， 事务要么是完全成功（ 提交时）或要么根本没有任何作用（回滚时）的工作单元。事务的不可分割（“原子”）属性是首字母缩写 ACID中的 “ A ”。

另请参见ACID，提交，回滚，事务。

```

## autocommit

A setting that causes a commit operation after each SQL statement. This mode is not recommended for working with InnoDB tables with transactions that span several statements. It can help performance for read-only transactions on InnoDB tables, where it minimizes overhead from locking and generation of undo data, especially in MySQL 5.6.4 and up. It is also appropriate for working with MyISAM tables, where transactions are not applicable.

See Also commit, locking, read-only transaction, SQL, transaction, undo.

```
自动提交

该设置会在每个SQL语句之后导致提交操作。不建议使用这种模式来处理跨多个语句的事务的InnoDB表。
它可以帮助InnoDB表执行只读事务，从而最大限度地减少了锁定和生成撤销数据的开销，特别是在MySQL 5.6.4及更高版本中.
对于不支持事务的MyISAM表来说，这也是合适的。

另请参阅提交，锁定，只读事务，SQL，事务，撤消。

```


## commit
A SQL statement that ends a transaction, making permanent any changes made by the transaction. It is the opposite of rollback, which undoes any changes made in the transaction.

InnoDB uses an optimistic mechanism for commits, so that changes can be written to the data files before the commit actually occurs. This technique makes the commit itself faster, with the tradeoff that more work is required in case of a rollback.

By default, MySQL uses the autocommit setting, which automatically issues a commit following each SQL statement.

See Also autocommit, optimistic, rollback, SQL, transaction.


```
提交

结束事务的SQL语句，永久保留事务所做的任何更改。它与回滚相反 ，它撤销了事务中所做的任何更改。

InnoDB使用提交的乐观机制，以便在提交实际发生之前将更改写入数据文件。这种技术使得提交本身的速度更快，在回滚的情况下需要做更多的工作。

默认情况下，MySQL使用 autocommit设置，该设置会在每个SQL语句之后自动发出提交。

另请参阅autocommit，optimistic，回滚，SQL，事务。

```

## locking

The system of protecting a transaction from seeing or changing data that is being queried or changed by other transactions. The locking strategy must balance reliability and consistency of database operations (the principles of the ACID philosophy) against the performance needed for good concurrency. Fine-tuning the locking strategy often involves choosing an isolation level and ensuring all your database operations are safe and reliable for that isolation level.

See Also ACID, concurrency, isolation level, locking, transaction.

```
锁定
保护事务免于查看或更改其他交易正在查询或更改的数据的系统 。
锁定策略必须平衡数据库操作（ACID原理的原则）的可靠性和一致性以及良好并发性所需的性能。
微调锁定策略通常涉及选择隔离级别并确保所有数据库操作对于该隔离级别安全可靠。

另请参阅ACID，并发性，隔离级别，锁定，事务。
```


## locking read
A SELECT statement that also performs a locking operation on an InnoDB table. Either SELECT ... FOR UPDATE or SELECT ... LOCK IN SHARE MODE. It has the potential to produce a deadlock, depending on the isolation level of the transaction. The opposite of a non-locking read. Not allowed for global tables in a read-only transaction.

SELECT ... FOR SHARE replaces SELECT ... LOCK IN SHARE MODE in MySQL 8.0.1, but LOCK IN SHARE MODE remains available for backward compatibility.

See Section 14.5.2.4, “Locking Reads”.

See Also deadlock, isolation level, locking, non-locking read, read-only transaction.

```
锁定读取
一个SELECT语句，它也对InnoDB表执行锁定操作。无论是SELECT ... FOR UPDATE或SELECT ... LOCK IN SHARE MODE。它有可能产生死锁，具体取决于事务的隔离级别。与非锁定读取相反。不允许在只读事务中使用全局表 。

SELECT ... FOR SHARE取代SELECT ... LOCK IN SHARE MODEMySQL 8.0.1，但 LOCK IN SHARE MODE仍可用于向后兼容。

请参见第14.5.2.4节“锁定读取”。

另请参阅死锁，隔离级别，锁定，非锁定读取，只读事务。
```


## non-locking read

A query that does not use the SELECT ... FOR UPDATE or SELECT ... LOCK IN SHARE MODE clauses. The only kind of query allowed for global tables in a read-only transaction. The opposite of a locking read. See Section 14.5.2.3, “Consistent Nonlocking Reads”.

SELECT ... FOR SHARE replaces SELECT ... LOCK IN SHARE MODE in MySQL 8.0.1, but LOCK IN SHARE MODE remains available for backward compatibility.

See Also locking read, query, read-only transaction.

```
不使用SELECT ... FOR UPDATE或SELECT ... LOCK IN SHARE MODE子句的查询。 唯一一种查询允许在只读事务中使用全局表。 与锁定读取相反。 请参见第14.5.2.3节“一致性非锁定读取”。

SELECT ... FOR SHARE替换MySQL 8.0.1中的SELECT ... LOCK IN SHARE MODE，但LOCK IN SHARE MODE仍然可用于向后兼容。

另请参阅锁定读取，查询，只读事务。
```


## read-only transaction
A type of transaction that can be optimized for InnoDB tables by eliminating some of the bookkeeping involved with creating a read view for each transaction. Can only perform non-locking read queries. It can be started explicitly with the syntax START TRANSACTION READ ONLY, or automatically under certain conditions. See Section 8.5.3, “Optimizing InnoDB Read-Only Transactions” for details.

See Also non-locking read, read view, transaction.

```
只读事务
通过消除为每个事务创建读取视图所涉及的一些簿记，可以针对InnoDB表优化的一类事务。 只能执行非锁定读取查询。 它可以使用语法START TRANSACTION READ ONLY来明确启动，或者在特定条件下自动启动。 有关详细信息，请参见第8.5.3节“优化InnoDB只读事务”。

另请参阅非锁定读取，读取视图，事务。

```


## row

The logical data structure defined by a set of columns. A set of rows makes up a table. Within InnoDB data files, each page can contain one or more rows.

Although InnoDB uses the term row format for consistency with MySQL syntax, the row format is a property of each table and applies to all rows in that table.

See Also column, data files, page, row format, table.
```
逻辑数据结构由一组列定义。一组行构成一张表。在InnoDB数据文件中，每个页面可以包含一行或多行。

虽然InnoDB使用术语行格式与MySQL语法保持一致，但行格式是每个表的属性，并适用于该表中的所有行。
```


## row lock
A lock that prevents a row from being accessed in an incompatible way by another transaction. Other rows in the same table can be freely written to by other transactions. This is the type of locking done by DML operations on InnoDB tables.

Contrast with table locks used by MyISAM, or during DDL operations on InnoDB tables that cannot be done with online DDL; those locks block concurrent access to the table.

See Also DDL, DML, InnoDB, lock, locking, online DDL, table lock, transaction.


