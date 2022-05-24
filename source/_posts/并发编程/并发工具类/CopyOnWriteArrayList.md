---
title: CopyOnWriteArrayList
categories: 
- 并发编程
- 并发工具类
---

CopyOnWrite 的意思是说，当容器需要被修改的时候，不直接修改当前容器，而是先将当前容器进行 Copy，复制出一个新的容器，然后修改新的容器，完成修改之后，再将原容器的引用指向新的容器。这样就完成了整个修改过程。

CopyOnWriteArrayList 的所有修改操作（add，set等）都是通过创建底层数组的新副本来实现的，所以 CopyOnWrite 容器也是一种**读写分离**的思想体现，读和写使用不同的容器。

**这个 List 具有以下特征：**

1. 线程安全的，多线程环境下可以直接使用，无需加锁；
2. 通过锁 + 数组拷贝 + volatile 关键字保证了线程安全；
3. 每次数组操作，都会把数组拷贝一份出来，在新数组上进行操作，操作成功之后再赋值回去。

**整体架构**

从整体架构上来说，CopyOnWriteArrayList 数据结构和 ArrayList 是一致的，底层是个数组，只不过 CopyOnWriteArrayList 在对数组进行操作的时候，基本会分四步走：

1. 加锁；
2. 从原数组中拷贝出新数组；
3. 在新数组上进行操作，并把新数组赋值给数组容器；
4. 解锁。

除了加锁之外，CopyOnWriteArrayList 的底层数组还被 volatile 关键字修饰，意思是一旦数组被修改，其它线程立马能够感知到，代码如下：

```java
private transient volatile Object[] array;
```

整体上来说，CopyOnWriteArrayList 就是利用锁 + 数组拷贝 + volatile 关键字保证了 List 的线程安全。

**适用场景**

在很多应用场景中，读操作可能会远远多于写操作。这种读多写少的场景也很适合使用 CopyOnWrite 集合。

**缺点**

这些缺点不仅是针对 CopyOnWriteArrayList，其实同样也适用于其他的 CopyOnWrite 容器

> 内存占用问题

因为 CopyOnWrite 的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，这一点会占用额外的内存空间。

> 在元素较多或者复杂的情况下，复制的开销很大

复制过程不仅会占用双倍内存，还需要消耗 CPU 等资源，会降低整体性能。

> 数据一致性问题

由于 CopyOnWrite 容器的修改是先修改副本，所以这次修改对于其他线程来说，并不是实时能看到的，只有在修改完之后才能体现出来。

如果你希望写入的的数据马上能被其他线程看到，CopyOnWrite 容器是不适用的。