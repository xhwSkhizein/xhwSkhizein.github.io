---
title: MysqlInnoDB事务隔离级别
layout: post
guid: urn:uuid:a1980d4a-b1c4-43b0-b19b-adeb6403ae8c
tags:
  - Mysql Isolation 事务 隔离 幻读 Phantom-reads 2PL
---

参考链接 :
- [Wikipedia_Isolation](https://en.wikipedia.org/wiki/Isolation_(database_systems))
- [Wikipedia_事务隔离](https://zh.wikipedia.org/wiki/%E4%BA%8B%E5%8B%99%E9%9A%94%E9%9B%A2)
- [Wikipedia_2PL](https://en.wikipedia.org/wiki/Two-phase_locking) （两阶段锁2PL，两阶段提交2PC）
- [Wikipedia_乐观并发控制](https://zh.wikipedia.org/wiki/%E4%B9%90%E8%A7%82%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6)
- [Oracle® Database Concepts, chapter 13 Data Concurrency and Consistency, Preventable Phenomena and Transaction Isolation Levels](http://docs.oracle.com/cd/B12037_01/server.101/b10743/consist.htm#sthref1919)


##### 事务的定义：
> 一个数据库事务通常包含了一个序列的对数据库的读/写操作。它的存在包含有以下两个目的：
1. 为数据库操作序列提供了一个从失败中恢复到正常状态的方法，同时提供了数据库即使在异常状态下仍能保持一致性的方法。
2. 当多个应用程序在并发访问数据库时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰。

> 白话版： 一串数据库操作指令，要么都成功，要没都失败，根据事务隔离级别事务间执行互不影响，一旦提交就永久保存在数据库里了

##### 事务的特性：ACID

* 原子性（Atomicity）
> 事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行

* 一致性（Consistency）
> 事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束

* 隔离性（Isolation）
> 多个事务并发执行时，一个事务的执行不应影响其他事务的执行

* 持久性（Durability）
> 已被提交的事务对数据库的修改应该永久保存在数据库中

##### 并发控制

> `并发控制`是数据库系统(以下改称·DBMS·)为了保证事务间的隔离和事务的准确执行所使用的一种低层实现机制。为了保证并行事务执行的准确执行，数据库和存储引擎在设计的时候着重强调了并发控制这一点。典型的事务并发控制机制是限制数据的访问顺序（执行调度）以满足`可序列化`和`可恢复性`。限制数据访问意味着降低了执行的性能，并发控制机制就是要保证在满足这些限制的前提下提供尽可能高的性能。经常在不损害正确性的情况下，为了达到更好的性能，可序列化的要求会减低一些，但是为了避免数据一致性的破坏，可恢复性必须保证。所以出现了事务隔离的级别，其实是针对并发控制程度和性能保障的让步。
> 数据库系统和存储引擎常用的并发控制机制有：[2PL](todo.com) 和 [MVCC](todo.com)

##### 事务隔离级别

> 因为DBMS中事务操作的`ACID`特性, 若要达到高级别的事务隔离机制，就需要DBMS使用`锁`或者 `MVCC`, 这样做就降低了并发行。
> 所以在很多数据库系统中，多数的数据库事务都避免高等级的隔离等级（如可序列化）从而减少对系统的锁定开销。我们需*要小心的分析数据库访问部分的代码来保证隔离级别的降低不会造成难以发现的代码bug*。相反的，更高的隔离级别会增加死锁发生的几率，同样需要编程过程中去避免。


* 可序列化(Serializable)

> 最高级别的事务隔离机制

* 可重复读(Repeatable reads)

* 提交读(Read committed)

* 未提交读(Read uncommitted)


##### 读现象

* 脏读
* 不可重复读
* 幻读


##### 隔离级别、读现象和锁































---
end
