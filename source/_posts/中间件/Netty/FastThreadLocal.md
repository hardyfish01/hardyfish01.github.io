---
title: FastThreadLocal
categories: 
- 中间件
- Netty
---

我们都有在源码中发现 FastThreadLocal 的身影。

Netty 作为高性能的网络通信框架，FastThreadLocal 是比 JDK 自身的 ThreadLocal 性能更高的通信框架。

FastThreadLocal 的实现与 ThreadLocal 非常类似，Netty 为 FastThreadLocal 量身打造了 FastThreadLocalThread 和 InternalThreadLocalMap 两个重要的类。

FastThreadLocalThread 是对 Thread 类的一层包装，每个线程对应一个 InternalThreadLocalMap 实例。

只有 FastThreadLocal 和 FastThreadLocalThread 组合使用时，才能发挥 FastThreadLocal 的性能优势。

首先看下 FastThreadLocalThread 的源码定义：

```java
public class FastThreadLocalThread extends Thread {
    private InternalThreadLocalMap threadLocalMap;
    // 省略其他代码
}
```

可以看出 FastThreadLocalThread 主要扩展了 InternalThreadLocalMap 字段，我们可以猜测到 FastThreadLocalThread 主要使用 InternalThreadLocalMap 存储数据，而不再是使用 Thread 中的 ThreadLocalMap。

FastThreadLocal 使用 Object 数组替代了 Entry 数组，`Object[0]` 存储的是一个`Set<FastThreadLocal<?>> `集合，从数组下标 1 开始都是直接存储的 value 数据，不再采用 ThreadLocal 的键值对形式进行存储。

![](https://img-blog.csdnimg.cn/93d5f3c827e7493e9950865920b3a956.png)

`InternalThreadLocalMap`源码：

<img src="https://img-blog.csdnimg.cn/a20cdc1e1eb74b618820c3a44282b645.png" style="zoom:25%;" />

 InternalThreadLocalMap 并没有使用线性探测法来解决 Hash 冲突，而是在 FastThreadLocal 初始化的时候分配一个数组索引 index，index 的值采用原子类 AtomicInteger 保证顺序递增，通过调用` InternalThreadLocalMap.nextVariableIndex() `方法获得。

然后在读写数据的时候通过数组下标 index 直接定位到 FastThreadLocal 的位置，时间复杂度为 O(1)。

如果数组下标递增到非常大，那么数组也会比较大，所以 FastThreadLocal 是通过空间换时间的思想提升读写性能。

 **FastThreadLocal.set() 的源码：**

<img src="https://img-blog.csdnimg.cn/aee8b6f703034386950ceefa2d67bcda.png" style="zoom:25%;" />

`InternalThreadLocalMap.get() `方法，源码如下：

<img src="https://img-blog.csdnimg.cn/2568000b93224efc9cba0ea1248b7e96.png" style="zoom:25%;" />

如果当前线程是 FastThreadLocalThread 类型，那么直接通过 fastGet() 方法获取 FastThreadLocalThread 的 threadLocalMap 属性即可。

slowGet() 是针对非 FastThreadLocalThread 类型的线程发起调用时的一种兜底方案。

如果当前线程不是 FastThreadLocalThread，内部是没有 InternalThreadLocalMap 属性的，Netty 在 UnpaddedInternalThreadLocalMap 中保存了一个 JDK 原生的 ThreadLocal，ThreadLocal 中存放着 InternalThreadLocalMap，此时获取 InternalThreadLocalMap 就退化成 JDK 原生的 ThreadLocal 获取。

**FastThreadLocal.get() 的源码实现如下：**

<img src="https://img-blog.csdnimg.cn/833bb86cc74e4d56869bedc6e2b0ee92.png" style="zoom:25%;" />

首先根据当前线程是否是 FastThreadLocalThread 类型找到 InternalThreadLocalMap，然后取出从数组下标 index 的元素，如果 index 位置的元素不是缺省对象 UNSET，说明该位置已经填充过数据，直接取出返回即可。

如果 index 位置的元素是缺省对象 UNSET，那么需要执行初始化操作。