---
title: CAS
categories: 
- 并发编程
---

**Synchronized**的方式加锁，会让线程在 BLOCKED 状态和 RUNNABLE 状态之间切换，在操作系统上，就会造成用户态和内核态的频繁切换，效率就比较低，所以产生一种CAS方式。

**CAS 是什么**

它的英文全称是 Compare-And-Swap，中文叫做比较并交换

* 有三个操作数：内存值 V、预期值 A、要修改的值 B。

* 仅当预期值 A 和当前的内存值 V 相同时，才将内存值修改为 B。

<img src="https://img-blog.csdnimg.cn/abcf0a039c5f41afb722b9561f98de5b.png" alt="img" style="zoom:25%;" />

**CAS有哪几个主要的缺点**

> ABA 问题

假设第一个线程拿到的初始值是 100，然后进行计算，在计算的过程中，有第二个线程把初始值改为了 200，然后紧接着又有第三个线程把 200 改回了 100。

等到第一个线程计算完毕去执行 CAS 的时候，它会比较当前的值是不是等于最开始拿到的初始值 100，此时会发现确实是等于 100，所以线程一就认为在此期间值没有被修改过，就理所当然的把这个 100 改成刚刚计算出来的新值，但实际上，在此过程中已经有其他线程把这个值修改过了，这样就会发生 ABA 问题。

**添加一个版本号就可以解决：**

我们在变量值自身之外，再添加一个版本号，那么这个值的变化路径就从 `A→B→A` 变成了 `1A→2B→3A`，这样一来，就可以通过对比版本号来判断值是否变化过，这比我们直接去对比两个值是否一致要更靠谱，所以通过这样的思路就可以解决 ABA 的问题了。

* 在 atomic 包中提供了 AtomicStampedReference 这个类，它是专门用来解决 ABA 问题的，解决思路正是利用版本号。

AtomicStampedReference 会维护一种类似 `<Object,int>` 的数据结构，其中的 int 就是用于计数的，也就是版本号，它可以对这个对象和 int 版本号同时进行原子更新，从而也就解决了 ABA 问题。

> 自旋时间过长

由于单次 CAS 不一定能执行成功，所以 CAS 往往是配合着循环来实现的，有的时候甚至是死循环，不停地进行重试，直到线程竞争不激烈的时候，才能修改成功。

可是如果我们的应用场景本身就是高并发的场景，就有可能导致 CAS 一直都操作不成功，这样的话，循环时间就会越来越长。而且在此期间，CPU 资源也是一直在被消耗的，这会对性能产生很大的影响。

所以这就要求我们，要根据实际情况来选择是否使用 CAS，在高并发的场景下，通常 CAS 的效率是不高的。

> 范围不能灵活控制

通常我们去执行 CAS 的时候，是针对某一个，而不是多个共享变量的，这个变量可能是 Integer 类型，也有可能是 Long 类型、对象类型等等，但是我们不能针对多个共享变量同时进行 CAS 操作，因为这多个变量之间是独立的，简单的把原子操作组合到一起，并不具备原子性。因此如果我们想对多个对象同时进行 CAS 操作并想保证线程安全的话，是比较困难的。

有一个解决方案，那就是利用一个新的类，来整合刚才这一组共享变量，这个新的类中的多个成员变量就是刚才的那多个共享变量，然后再利用 atomic 包中的 AtomicReference 来把这个新对象整体进行 CAS 操作，这样就可以保证线程安全。