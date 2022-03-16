---
title: TreeMap
categories: 
- Java编程思想
---

TreeMap 底层的数据结构就是红黑树，和 HashMap 的红黑树结构一样

* 不同的是，TreeMap 利用了红黑树左节点小，右节点大的性质，根据 key 进行排序，使每个元素能够插入到红黑树大小适当的位置，维护了 key 的大小关系，适用于 key 需要排序的场景。

因为底层使用的是平衡红黑树的结构，所以 containsKey、get、put、remove 等方法的时间复杂度都是`log(n)`

**TreeMap 常见的属性有：**

```java
//比较器，如果外部有传进来 Comparator 比较器，首先用外部的
//如果外部比较器为空，则使用 key 自己实现的 Comparable#compareTo 方法
//比较手段和上面日常工作中的比较 demo 是一致的
private final Comparator<? super K> comparator;
 
//红黑树的根节点
private transient Entry<K,V> root;
 
//红黑树的已有元素大小
private transient int size = 0;
 
//树结构变化的版本号，用于迭代过程中的快速失败场景
private transient int modCount = 0;
 
//红黑树的节点
static final class Entry<K,V> implements Map.Entry<K,V> {}
```

**新增节点**

我们来看下 TreeMap 新增节点的步骤：

1.判断红黑树的节点是否为空，为空的话，新增的节点直接作为根节点

```java
Entry<K,V> t = root;
//红黑树根节点为空，直接新建
if (t == null) {
    // compare 方法限制了 key 不能为 null
    compare(key, key); // type (and possibly null) check
    // 成为根节点
    root = new Entry<>(key, value, null);
    size = 1;
    modCount++;
    return null;
}
```

2.根据红黑树左小右大的特性，进行判断，找到应该新增节点的父节点：

```java
Comparator<? super K> cpr = comparator;
if (cpr != null) {
    //自旋找到 key 应该新增的位置，就是应该挂载那个节点的头上
    do {
        //一次循环结束时，parent 就是上次比过的对象
        parent = t;
        // 通过 compare 来比较 key 的大小
        cmp = cpr.compare(key, t.key);
        //key 小于 t，把 t 左边的值赋予 t，因为红黑树左边的值比较小，循环再比
        if (cmp < 0)
            t = t.left;
        //key 大于 t，把 t 右边的值赋予 t，因为红黑树右边的值比较大，循环再比
        else if (cmp > 0)
            t = t.right;
        //如果相等的话，直接覆盖原值
        else
            return t.setValue(value);
        // t 为空，说明已经到叶子节点了
    } while (t != null);
}
```

3.在父节点的左边或右边插入新增节点

```java
//cmp 代表最后一次对比的大小，小于 0 ，代表 e 在上一节点的左边
if (cmp < 0)
    parent.left = e;
//cmp 代表最后一次对比的大小，大于 0 ，代表 e 在上一节点的右边，相等的情况第二步已经处理了。
else
    parent.right = e;
```

4.着色旋转，达到平衡，结束。

从源码中，我们可以看到：

1. 新增节点时，就是利用了红黑树左小右大的特性，从根节点不断往下查找，直到找到节点是 null 为止，节点为 null 说明到达了叶子结点；
2. 查找过程中，发现 key 值已经存在，直接覆盖；
3. TreeMap 是禁止 key 是 null 值的。