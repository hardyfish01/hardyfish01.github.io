---
title: TreeSet
categories: 
- Java编程思想
---

TreeSet 大致的结构和 HashSet 相似，底层组合的是 TreeMap，所以继承了 TreeMap key 能够排序的功能，迭代的时候，也可以按照 key 的排序顺序进行迭代。

 **TreeSet 组合 TreeMap 实现的两种思路：**

1. TreeSet 直接使用 TreeMap 的某些功能，自己包装成新的 api。
2. TreeSet 定义自己想要的 api，自己定义接口规范，让 TreeMap 去实现。

```java
// NavigableSet 接口，定义了迭代的一些规范，和一些取值的特殊方法
// TreeSet 实现了该方法，也就是说 TreeSet 本身已经定义了迭代的规范
public interface NavigableSet<E> extends SortedSet<E> {
    Iterator<E> iterator();
    E lower(E e);
}  
// m.navigableKeySet() 是 TreeMap 写了一个子类实现了 NavigableSet
// 接口，实现了 TreeSet 定义的迭代规范
public Iterator<E> iterator() {
    return m.navigableKeySet().iterator();
}
```

**常见面试题**

> 如果我想实现根据 key 的新增顺序进行遍历怎么办？

要按照 key 的新增顺序进行遍历，首先想到的应该就是 LinkedHashMap，而 LinkedHashSet 正好是基于 LinkedHashMap 实现的，所以我们可以选择使用 LinkedHashSet。

> 如果我想对 key 进行去重，有什么好的办法么？

我们首先想到的是 TreeSet，TreeSet 底层使用的是 TreeMap，TreeMap 在 put 的时候，如果发现 key 是相同的，会把 value 值进行覆盖，所有不会产生重复的 key ，利用这一特性，使用 TreeSet 正好可以去重。