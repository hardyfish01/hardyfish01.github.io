---
title: 认识KAFKA
categories: 
- Apache KAFKA实战
---

Apache Kafka是由LinkedIn采用Scala和Java开发的开源流处理软件平台，并捐赠给了Apache Software Foundation。

> 该项目旨在提供统一的、高吞吐量、低延迟的平台来处理实时数据流。

Kafka可以通过Kafka Connect连接到外部系统，并提供了Kafka Streams。

**Kafka的特性**

Kafka是一种分布式的，基于发布/订阅的消息系统，主要特性如下：

| 特性              | 分布式           | **高性能**       | **持久性和扩展性** |
| :---------------- | :--------------- | :--------------- | :----------------- |
| 描述              | 多分区           | 高吞吐量         | 数据可持久化       |
| 多副本            | 低延迟           | 容错性           |                    |
| 多订阅者          | 高并发           | 支持水平在线扩展 |                    |
| 基于ZooKeeper调度 | 时间复杂度为O(1) | 消息自动平衡     |                    |

**Kafka版本命名**

我们在官网上下载Kafka时，会看到这样的版本：

![](https://img-blog.csdnimg.cn/756622272bf847158a62e6661458c77e.png)

前面的版本号是编译Kafka源代码的Scala编译器版本。

Kafka服务器端的代码完全由Scala语言编写，Scala同时支持面向对象编程和函数式编程，用Scala写成的源代码编译之后也是普通的`.class`文件，因此我们说Scala是JVM系的语言。

真正的Kafka版本号实际上是`2.2.1`。

> 那么这个2.2.1又表示什么呢？

前面的2表示大版本号，即Major Version；中间的2表示小版本号或次版本号，即Minor Version；最后的1表示修订版本号，也就是Patch号。

Kafka社区在发布1.0.0版本后写过一篇文章，宣布Kafka版本命名规则正式从4位演进到3位，比如0.11.0.0版本就是4位版本号。

> 有个建议，不论用的是哪个版本，都请尽量保持服务器端版本和客户端版本一致，否则你将损失很多Kafka为你提供的性能优化收益。

**版本演进**

0.7版本：只提供了最基础的消息队列功能。

0.8版本：引入了副本机制，至此kafka成为了一个整整意义上完备的分布式可靠消息队列解决方案

0.9.0.0版本：增加了基础的安全认证/权限功能；使用Java重新了新版本消费者API；引入了Kafka Connect组件。

0.11.0.0版本：提供了幂等性Producer API以及事务API；对Kafka消息格式做了重构。

1.0和2.0版本：主要还是Kafka Streams的各种改进

<img src="https://img-blog.csdnimg.cn/d29df08998f54eb68e0daa68db632bc0.png" style="zoom:25%;" />

**kafka设计思想**

一个最基本的架构是生产者发布一个消息到Kafka的一个Topic ，该Topic的消息存放于的Broker中，消费者订阅这个Topic，然后从Broker中消费消息，下面这个图可以更直观的描述这个场景：

![](https://img-blog.csdnimg.cn/2a527d49305b4879a68a4eb753f571ba.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

**消息状态：**在Kafka中，消息是否被消费的状态保存在Consumer中，Broker不会关心消息是否被消费或被谁消费，Consumer会记录一个offset值（指向partition中下一条将要被消费的消息位置），如果offset被错误设置可能导致同一条消息被多次消费或者消息丢失。

**消息持久化：**Kafka会把消息持久化到本地文件系统中，并且具有极高的性能。

**批量发送：**Kafka支持以消息集合为单位进行批量发送，以提高效率。

**Push-and-Pull：**Kafka中的Producer和Consumer采用的是Push-and-Pull模式，即Producer向Broker Push消息，Consumer从Broker Pull消息。

**分区机制（Partition）：**Kafka的Broker端支持消息分区，Producer可以决定把消息发到哪个Partition，在一个Partition中消息的顺序就是Producer发送消息的顺序，一个Topic中的Partition数是可配置的，Partition是Kafka高吞吐量的重要保证。

**系统架构**

![](https://img-blog.csdnimg.cn/c9f0b1f3bd924aed9e67e994926c996f.png)

通常情况下，一个kafka体系架构包括**多个Producer**、**多个Consumer**、**多个broker**以及**一个Zookeeper集群**。

**Producer**：生产者，负责将消息发送到kafka中。

**Consumer**：消费者，负责从kafka中拉取消息进行消费。

**Broker**：Kafka服务节点，一个或多个Broker组成了一个Kafka集群

**Zookeeper集群**：负责管理kafka集群元数据以及控制器选举等。

# 高性能原因

**顺序读写**

* kafka的消息是不断追加到文件中的，这个特性使`kafka`可以充分利用磁盘的顺序读写性能

* 顺序读写不需要硬盘磁头的寻道时间，只需很少的扇区旋转时间，所以速度远快于随机读写

Kafka 可以配置异步刷盘，不开启同步刷盘，异步刷盘不需要等写入磁盘后返回消息投递的 ACK，所以它提高了消息发送的吞吐量，降低了请求的延时

**零拷贝**

传统的 IO 流程，需要先把数据拷贝到内核缓冲区，再从内核缓冲拷贝到用户空间，应用程序处理完成以后，再拷贝回内核缓冲区

这个过程中发生了多次数据拷贝

* 为了减少不必要的拷贝，`Kafka` 依赖 Linux 内核提供的 `Sendfile` 系统调用

在 Sendfile 方法中，数据在内核缓冲区完成输入和输出，不需要拷贝到用户空间处理，这也就避免了重复的数据拷贝

在具体的操作中，Kafka 把所有的消息都存放在单独的文件里，在消息投递时直接通过 `Sendfile` 方法发送文件，减少了上下文切换，因此大大提高了性能

**MMAP技术**

除了 `Sendfile` 之外，还有一种零拷贝的实现技术，即 Memory Mapped Files

Kafka 使用 `Memory Mapped Files` 完成内存映射，`Memory Mapped Files` 对文件的操作不是 `write/read`，而是直接对内存地址的操作，如果是调用文件的 `read` 操作，则把数据先读取到内核空间中，然后再复制到用户空间，但 `MMAP`可以将文件直接映射到用户态的内存空间，省去了用户空间到内核空间复制的开销

* Producer生产的数据持久化到broker，采用mmap文件映射，实现顺序的快速写入

* Customer从broker读取数据，采用sendfile，将磁盘文件读到OS内核缓冲区后，直接转到socket buffer进行网络发送。

**批量发送读取**

* Kafka 的批量包括批量写入、批量发布等。它在消息投递时会将消息缓存起来，然后批量发送

* 同样，消费端在消费消息时，也不是一条一条处理的，而是批量进行拉取，提高了消息的处理速度

**数据压缩**

Kafka还支持对消息集合进行压缩，`Producer`可以通过`GZIP`或`Snappy`格式对消息集合进行压缩

* 压缩的好处就是减少传输的数据量，减轻对网络传输的压力

Producer压缩之后，在`Consumer`需进行解压，虽然增加了CPU的工作，但在对大数据处理上，瓶颈在网络上而不是CPU，所以这个成本很值得

**分区**

kafka中的topic中的内容可以被分为多partition存在，每个partition又分为多个段segment，所以每次操作都是针对一小部分做操作，很轻便，并且增加`并行操作`的能力