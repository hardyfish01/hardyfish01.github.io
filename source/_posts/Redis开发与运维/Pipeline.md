---
title: Pipeline
categories: 
- Redis开发与运维
---

[Redis如何解决频繁的命令往返造成的性能瓶颈！](https://mp.weixin.qq.com/s/HDBD5iUwzrMGT9mscTARTA)

Redis提供了批量操作命令(例如mget、mset等)，有效地节约RTT。

但大部分命令是不支持批量操作的，例如要执行n次hgetall命令，并没有 mhgetall命令存在，需要消耗n次RTT。

Pipeline(流水线)机制它能将一组Redis命令进行组装，通过一次RTT传输给Redis，再将这组Redis命令的执行结果按顺序返回给客户端。

使用Pipeline执行了n次命令，整个过程需要1次RTT。

<img src="https://img-blog.csdnimg.cn/0952e4e99e10435c9aa30a3a4bad3078.png" style="zoom:25%;" />

**原生批量命令与Pipeline对比** 

具体包含以下几点: 

* 原生批量命令是原子的，Pipeline是非原子的。 

* 原生批量命令是一个命令对应多个key，Pipeline支持多个命令。

* 原生批量命令是Redis服务端支持实现的，而Pipeline需要服务端和客户端的共同实现。

**最佳实践**

Pipeline虽然好用，但是每次Pipeline组装的命令个数不能没有节制，否 则一次组装Pipeline数据量过大，一方面会增加客户端的等待时间，另一方 面会造成一定的网络阻塞，可以将一次包含大量命令的Pipeline拆分成多次较小的Pipeline来完成。

Pipeline只能操作一个Redis实例，但是即使在分布式Redis场景中，也可以作为批量操作的重要优化手段。