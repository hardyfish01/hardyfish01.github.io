---
title: HashSet
categories: 
- 编程语言
- Java基础
- 集合类
---

**HashSet简介**

1. HashSet实现了Set接口
2. HashSet实际上是HashMap
3. 可以存放null值，但是只能有一个null
4. HashSet不保证元素是有序的，取决于hash后，在确定索引的结果（即不保证存放元素的顺序和取出的顺序一致）
5. 不能有重复元素/对象

**HashSet定义**

```java
public class HashSet<E>
     extends AbstractSet<E>
     implements Set<E>, Cloneable, java.io.Serializable
```

**HashSet 是如何组合 HashMap 的**

HashSet 的实现是基于 HashMap 的

```java
// 把 HashMap 组合进来，key 是 Hashset 的 key，value 是下面的 PRESENT
private transient HashMap<E,Object> map;
// HashMap 中的 value
private static final Object PRESENT = new Object();
```

从这两行代码中，我们可以看出两点：

1. 我们在使用 HashSet 时，比如 add 方法，只有一个入参，但组合的 Map 的 add 方法却有 key，value 两个入参，相对应上 Map 的 key 就是我们 add 的入参，value 就是第二行代码中的 PRESENT；
2. 如果 HashSet 是被共享的，当多个线程访问的时候，就会有线程安全问题，因为在后续的所有操作中，并没有加锁。

**初始化**

HashSet 的初始化比较简单，直接 new HashMap 即可，当有原始集合数据进行初始化的情况下，会对 HashMap 的初始容量进行计算，源码如下：

```java
// 对 HashMap 的容量进行了计算
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
```

上述代码中：`Math.max ((int) (c.size ()/.75f) + 1, 16)`，就是对 HashMap 的容量进行了计算，翻译成中文就是 取括号中两个数的最大值（`期望的值 / 0.75+1`，默认值 16）。

至于 HashSet 的其他方法就比较简单了，就是对 Map 的 API 进行了一些包装，如下的 add 方法实现：

```java
public boolean add(E e) {
    // 直接使用 HashMap 的 put 方法，进行一些简单的逻辑判断
    return map.put(e, PRESENT)==null;
}
```