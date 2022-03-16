---
title: HashSet
categories: 
- Java编程思想
---

底层实现基于 HashMap，所以迭代时不能保证按照插入顺序，或者其它顺序进行迭代；

add、remove、contanins、size 等方法的耗时性能，是不会随着数据量的增加而增加的，这个主要跟 HashMap 底层的数组数据结构有关，不管数据量多大，不考虑 hash 冲突的情况下，时间复杂度都是 O(1)；

**基本特点：**

* 线程不安全的，如果需要安全请自行加锁，或者使用 `Collections.synchronizedSet`；

* 迭代过程中，如果数据结构被改变，会快速失败的，会抛出 ConcurrentModificationException 异常；

**HashSet是如何组合HashMap的**

```java
// 把 HashMap 组合进来，key 是 Hashset 的 key，value 是下面的 PRESENT
private transient HashMap<E,Object> map;
// HashMap 中的 value
private static final Object PRESENT = new Object();
```

注意两点：

1. 我们在使用 HashSet 时，比如 add 方法，只有一个入参，但组合的 Map 的 add 方法却有 key，value 两个入参，相对应上 Map 的 key 就是我们 add 的入参，value 就是第二行代码中的 PRESENT，此处设计非常巧妙，用一个默认值 PRESENT 来代替 Map 的 Value；
2. 如果 HashSet 是被共享的，当多个线程访问的时候，就会有线程安全问题，因为在后续的所有操作中，并没有加锁。

 **add 方法实现：**

```java
public boolean add(E e) {
    // 直接使用 HashMap 的 put 方法，进行一些简单的逻辑判断
    return map.put(e, PRESENT)==null;
}
```

**初始化**

```java
// 对 HashMap 的容量进行了计算
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
```

上述代码中：`Math.max ((int) (c.size ()/.75f) + 1, 16)`，就是对 HashMap 的容量进行了计算，翻译成中文就是取括号中两个数的最大值（`期望的值 / 0.75+1，默认值 16`）：

1. 和 16 比较大小的意思是说，如果给定 HashMap 初始容量小于 16 ，就按照 HashMap 默认的 16 初始化好了，如果大于 16，就按照给定值初始化。
2. HashMap 扩容的计算公式是：`Map 的容量 * 0.75f`，一旦达到阀值就会扩容，此处用` (int) (c.size ()/.75f) + 1 `来表示初始化的值，这样使我们期望的大小值正好比扩容的阀值还大 1，就不会扩容，符合 HashMap 扩容的公式。