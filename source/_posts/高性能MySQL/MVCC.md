---
title: MVCC
categories: 
- 高性能MySQL
---

MVCC多版本并发控制机制最大的好处是读不加锁，读写不冲突

**快照读与当前读**

在 MVCC 并发控制中，读操作可以分为两类: 快照读（Snapshot Read）与当前读 （Current Read）。

* 快照读：读取的是记录的可见版本（有可能是历史版本），不用加锁。

* 当前读：读取的是记录的最新版本，并且当前读返回的记录，都会加锁，保证其他事务不会再并发修改这条记录。 

> 注意：MVCC 只在 Read Commited 和 Repeatable Read 两种隔离级别下工作。

* 快照读：简单的 select 操作，属于快照读，不需要加锁。 

* 当前读：特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。 

[MVCC由浅入深学习](https://mp.weixin.qq.com/s/jxM7n_4Or52_-MlB4aK9Bw)