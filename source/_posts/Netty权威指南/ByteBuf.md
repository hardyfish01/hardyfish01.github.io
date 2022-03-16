---
title: ByteBuf
categories: 
- Netty权威指南
---

ByteBuf 是 Netty 的数据容器，所有网络通信中字节流的传输都是通过 ByteBuf 完成的。

**ByteBuf 的结构：**

<img src="https://img-blog.csdnimg.cn/70dca391870c46b886a17050e9536e2f.png" style="zoom:25%;" />

从上面这幅图可以看到：

1. ByteBuf 是一个字节容器，容器里面的的数据分为三个部分，第一个部分是已经丢弃的字节，这部分数据是无效的；
2. 第二部分是可读字节，这部分数据是 ByteBuf 的主体数据，从 ByteBuf 里面读取的数据都来自这一部分;
3. 最后一部分的数据是可写字节，所有写到 ByteBuf 的数据都会写到这一段。
4. 最后一部分虚线表示的是该 ByteBuf 最多还能扩容多少容量 

从 ByteBuf 中每读取一个字节，readerIndex 自增1，ByteBuf 里面总共有 `writerIndex-readerIndex` 个字节可读，由此可以推论出当 readerIndex 与 writerIndex 相等的时候，ByteBuf 不可读。

写数据是从 writerIndex 指向的部分开始写，每写一个字节，writerIndex 自增1，直到增到 capacity，这个时候，表示 ByteBuf 已经不可写了。

ByteBuf 里面其实还有一个参数 maxCapacity，当向 ByteBuf 写数据的时候，如果容量不足，那么这个时候可以进行扩容，直到 capacity 扩容到 maxCapacity，超过 maxCapacity 就会报错。

**引用计数**

ByteBuf 是基于引用计数设计的，它实现了 ReferenceCounted 接口，ByteBuf 的生命周期是由引用计数所管理。

只要引用计数大于 0，表示 ByteBuf 还在被使用；当 ByteBuf 不再被其他对象所引用时，引用计数为 0，那么代表该对象可以被释放。

引用计数对于 Netty 设计缓存池化有非常大的帮助，当引用计数为 0，该 ByteBuf 可以被放入到对象池中，避免每次使用 ByteBuf 都重复创建，对于实现高性能的内存管理有着很大的意义。

此外 Netty 可以利用引用计数的特点实现内存泄漏检测工具。

JVM 并不知道 Netty 的引用计数是如何实现的，当 ByteBuf 对象不可达时，一样会被 GC 回收掉，但是如果此时 ByteBuf 的引用计数不为 0，那么该对象就不会释放或者被放入对象池，从而发生了内存泄漏。

**ByteBuf 分类**

Heap/Direct 就是堆内和堆外内存。Heap 指的是在 JVM 堆内分配，底层依赖的是字节数据；Direct 则是堆外内存，不受 JVM 限制，分配方式依赖 JDK 底层的 ByteBuffer。

Pooled/Unpooled 表示池化还是非池化内存。Pooled 是从预先分配好的内存中取出，使用完可以放回 ByteBuf 内存池，等待下一次分配。而 Unpooled 是直接调用系统 API 去申请内存，确保能够被 JVM GC 管理回收。

Unsafe/非 Unsafe 的区别在于操作方式是否安全。 Unsafe 表示每次调用 JDK 的 Unsafe 对象操作物理内存，依赖 offset + index 的方式操作数据。非 Unsafe 则不需要依赖 JDK 的 Unsafe 对象，直接通过数组下标的方式操作数据。

**ByteBuffer类的操作**

> 分配缓冲区

ByteBuf 分配一个缓冲区，仅仅给定一个初始值就可以。

```java
ByteBuf buf = Unpooled.buffer(13);
System.out.println(String.format("init: ridx=%s widx=%s cap=%s", buf.readerIndex(), buf.writerIndex(), buf.capacity()));
```

> 写操作

ByteBuf 写操作和 ByteBuffer 类似，只是写指针是单独记录的，ByteBuf 的写操作支持多种类型。

写入字节数组类型：

```java
String content = "月伴飞鱼公众号";
buf.writeBytes(content.getBytes());
System.out.println(String.format("write: ridx=%s widx=%s cap=%s", buf.readerIndex(), buf.writerIndex(), buf.capacity()));
```

> 读操作

一样的，ByteBuf 写操作和 ByteBuffer 类似，只是写指针是单独记录的，ByteBuf 的读操作支持多种类型。

从当前 readerIndex 位置读取四个字节内容：

```java
byte[] dst = new byte[4];
buf.readBytes(dst);
System.out.println(new String(dst));
System.out.println(String.format("read(4): ridx=%s widx=%s cap=%s", buf.readerIndex(), buf.writerIndex(), buf.capacity()));
```
