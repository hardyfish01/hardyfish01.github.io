---
title: GC日志
categories: 
- JVM相关
---

**GC 事件的类型简介**

**Minor GC（小型 GC）**

收集年轻代内存的 GC 事件称为 Minor GC。

1. 当 JVM 无法为新对象分配内存空间时就会触发 Minor GC（ 一般就是 Eden 区用满了）。如果对象的分配速率很快，那么 Minor GC 的次数也就会很多，频率也就会很快。
2. Minor GC 事件不处理老年代，所以会把所有从老年代指向年轻代的引用都当做 GC Root。从年轻代指向老年代的引用则在标记阶段被忽略。
3. 与我们一般的认知相反，Minor GC 每次都会引起 STW 停顿（stop-the-world），挂起所有的应用线程。对大部分应用程序来说，Minor GC 的暂停时间可以忽略不计，因为 Eden 区里面的对象大部分都是垃圾，也不怎么复制到存活区/老年代。但如果不符合这种情况，那么很多新创建的对象就不能被 GC 清理，Minor GC 的停顿时间就会增大，就会产生比较明显的 GC 性能影响。

> 简单定义：Minor GC 清理的是年轻代，又或者说 Minor GC 就是“年轻代 GC”（Young GC，简称 YGC）。

**Major GC vs. Full GC**

- Major GC（大型 GC）：清理老年代空间（Old Space）的 GC 事件。
- Full GC（完全 GC）：清理整个堆内存空间的 GC 事件，包括年轻代空间和老年代空间。

```
正在执行...
[GC (Allocation Failure)
  [PSYoungGen: 65081K->10728K(76288K)]
  65081K->27102K(251392K), 0.0112478 secs]
  [Times: user=0.03 sys=0.02, real=0.01 secs]
......此处省略了多行
[Full GC (Ergonomics)
  [PSYoungGen: 80376K->0K(872960K)]
  [ParOldGen: 360220K->278814K(481280K)]
  440597K->278814K(1354240K),
  [Metaspace: 3443K->3443K(1056768K)],
  0.0406179 secs]
  [Times: user=0.23 sys=0.01, real=0.04 secs]
执行结束!共生成对象次数:746
Heap
 PSYoungGen total 872960K, used 32300K [0x000000076ab00000, 0x00000007b0180000, 0x00000007c0000000)
  eden space 792576K, 4% used [0x000000076ab00000,0x000000076ca8b370,0x000000079b100000)
  from space 80384K, 0% used [0x00000007a3800000,0x00000007a3800000,0x00000007a8680000)
  to space 138240K, 0% used [0x000000079b100000,0x000000079b100000,0x00000007a3800000)
 ParOldGen total 481280K, used 278814K [0x00000006c0000000, 0x00000006dd600000, 0x000000076ab00000)
  object space 481280K, 57% used [0x00000006c0000000,0x00000006d1047b10,0x00000006dd600000)
 Metaspace used 3449K, capacity 4494K, committed 4864K, reserved 1056768K
  class space used 366K, capacity 386K, committed 512K, reserved 1048576K
```

**Heap 堆内存使用情况**

```java
PSYoungGen total 872960K, used 32300K [0x......)
 eden space 792576K, 4% used [0x......)
 from space 80384K, 0% used [0x......)
 to space 138240K, 0% used [0x......)
```

PSYoungGen，年轻代总计 872960K，使用量 32300K，后面的方括号中是内存地址信息。

- 其中 eden space 占用了 792576K，其中 4% used
- 其中 from space 占用了 80384K，其中 0% used
- 其中 to space 占用了 138240K，其中 0% used

```java
ParOldGen total 481280K, used 278814K [0x......)
 object space 481280K, 57% used [0x......)
```

ParOldGen，老年代总计 total 481280K，使用量 278814K。

- 其中 object space 占用了 481280K，其中 57% used

```java
Metaspace used 3449K, capacity 4494K, committed 4864K, reserved 1056768K
 class space used 366K, capacity 386K, committed 512K, reserved 1048576K
```

Metaspace，元数据区总计使用了 3449K，容量是 4494K，JVM 保证可用的大小是 4864K，保留空间 1GB 左右。

- 其中 class space 使用了 366K，capacity 386K

