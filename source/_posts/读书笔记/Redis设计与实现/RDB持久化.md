---
title: RDB持久化
categories: 
- 读书笔记
- Redis设计与实现
---

由于Redis是内存数据库，数据状态都存储于内存，如果不想办法将存储在内存中的数据库状态保存到磁盘里，那么一旦服务器进程退出，服务器中的数据库状态也会消失。

为解决这个问题，Redis提供了持久化的功能，可将内存中的数据库保存到磁盘，防止意外丢失。

* RDS持久化（默认持久化策略）就是将某一时间点上的状态保存到一个RDB文件里。

RDB文件是经过**压缩的二进制文件**，可通过该文件还原成数据库状态。

**RDB文件的创建与载入**

有两个命令可用于生成RDB文件（SAVE和BGSAVE）。

他们之间的区别是：

* SAVE会**阻塞**Redis服务器进程，直到RDB文件创建完毕为止，阻塞期间，服务器不能处理任何命令请求。

* BGSAVE会**fork出一个子进程**，由子进程负责创建RDB文件，**父进程继续处理命令请求**。当子进程完成之后，向父进程**发送信号**。

创建就是执行SAVE/BGSAVE底层调用rdbSave函数的过程，载入就是服务启动时读取RDB文件底层调用rdbLoad函数的过程。

**适用场景**

- 适合大规模的数据恢复
- 对数据完整性和一致性要求不高

**缺陷**

- 在一定间隔时间做一次备份，所以如果redis挂了，就会**丢失最后一次快照后的所有修改**。
- fork的时候，内存中的数据被克隆一份，大致**2倍的膨胀性**需要考虑内存空间。

**RDB与AOF共存的载入情况**

RDB文件的载入是在服务器启动时执行，Redis并没有专门提供载入Redis的命令。

由于AOF文件的更新频率更高，因此开启AOF持久化功能后，启动时**优先加载AOF**还原数据，只有在AOF处于关闭状态，才使用RDB文件恢复数据。

**自动间隔性保存**

服务器允许用户通过配置文件设置隔一定时间自动执行BGSAVE。

可通过save选项设多个保存条件，默认的配置如下：

```text
save 900 1
save 300 10
save 60 10000
```

只要满足任意条件，900s内对数据库进行1次修改BGSAVE就会被执行。

**分析RDB文件**

Redis自带RDB文件检查工具redis-check-dump。可以帮助在系统故障后分析快照文件，也就是RDB文件。