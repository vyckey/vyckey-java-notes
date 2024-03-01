---
title: MySQL Transaction
tags: [mysql, transaction]
sidebar_label: Transaction
sidebar_position: 9
---

# MySQL Transaction

## ACID特性

* **原子性**（`Atomicity`） ： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
* **一致性**（`Consistency`）： 执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；
* **隔离性**（`Isolation`）： 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的。
* **持久性**（`Durability`）： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

## 事务分类

* 扁平事务
* 链事务
* 嵌套事务
* 分布式事务

## 事务隔离级别

* **读取未提交**（`READ-UNCOMMITTED`）： 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
* **读取已提交**（`READ-COMMITTED`）： 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
* **可重复读**（`REPEATABLE-READ`）： 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
* **可串行化**（`SERIALIZABLE`）： 最高的隔离级别，完全服从`ACID`的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :--- | :---: | :---: | :---: |
| READ-UNCOMMITTED | √ | √ | √ |
| READ-COMMITTED | × | √ | √ |
| REPEATABLE-READ | × | × | √ |
| SERIALIZABLE | × | × | × |

InnoDB存储引擎默认支持的隔离级别是**REPEATABLE-READ**，但与标准SQL不同的是，InnoDB存储引擎在REPEATABLE-READ的事务隔离级别下，使用**Next-Key Lock锁算法**，**避免幻读的产生**。也就是说其通过其他方式已经达到了SERIALIZABLE的标准。

# 参考资料