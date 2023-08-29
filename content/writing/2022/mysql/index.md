---
title: "MySQL"
subtitle: ""
date: 2022-09-29T13:59:02+08:00
lastmod: 2022-09-29T13:59:02+08:00
draft: false
toc: true
comments: true
tags:
  - Database
  - MySQL
---

## MySQL, Oracle，SQL Server 的区别
1. Oracle 没有自动增长类型，MySQL 和 SQL Server 一般使用自动增长类型

2. 做分页的话，MySQL 使用 Limit，SQL Server 使用 top，Oracle 使用 row

3. Oracle 支持多用户不同权限来进行操作，而 MySQL 只要有登录权限就可操作全部数据库

## 数据库三大范式
第一范式：每个列都不可以再拆分。

第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。

第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。

## Mysql 事务
事务 (transaction) 是作为一个单元的一组有序的数据库操作 (sql 语句分组)。

## 事务的四大特性 (ACID)
1. 原子性 (Atomicity)：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；

2. 一致性 (Consistency)：执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的；

3. 隔离性 (isolation)：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；

4. 持久性 (Durability)：一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

## 什么是脏读？幻读？不可重复读？
1. 脏读：某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个 RollBack 了操作，则后一个事务所读取的数据就会是不正确的。

2. 不可重复读：不可重复读 (Non-repeatable read): 在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。

3. 幻读：在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列 (Row) 数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有几列数据是它先前所没有的。

## 事务的隔离级别
SQL 标准定义了 4 类隔离级别，包括了一些具体规则，用来限定事务内外的哪些改变是可见的，哪些是不可见的。

1. 读取未提交内容 (Read Uncommitted)：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。

2. 读取提交内容 (Read Committed)：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。

3. 可重复读 (Repeatable Read)：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

4. 串行读 (Serializable)：最高的隔离级别。通过强制事务排序，使之不能互相冲突，从而解决幻读问题。简言之，在每个读的数据行上加上共享锁。在这个级别时，可能导致大量的超时现象和锁竞争。

## SQL 语句
### 六种关联查询
1. 交叉连接

2. 内连接

3. 外连接

4. 联合查询

5. 全连接

6. 交叉连接

## SQL 优化
1. 选取最适合的字段属性
例如 在定义邮政编码这个字段的时候，如果设置为 char (255)，就会给数据库增加不必要的空间，甚至使用 varchar 这种类型也是多余的，char (6) 就够了

2. 尽量把字段设置为 not null
这样执行查询时，数据库不会去比较 Null 值

3. 优化查询语句

4. 使用索引
查询时使用 join 代替子查询
尽量使用一条或者少数几条语句完成

5. 使用外键