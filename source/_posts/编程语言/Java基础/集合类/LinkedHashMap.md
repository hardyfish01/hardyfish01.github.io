---
title: LinkedHashMap
categories: 
- 编程语言
- Java基础
- 集合类
---

**LinkedHashMap 整体架构**

LinkedHashMap 本身是继承 HashMap 的，所以它拥有 HashMap 的所有特性，再此基础上，还提供了两大特性：

- 按照插入顺序进行访问；
- 实现了访问最少最先删除功能，其目的是把很久都没有访问的 key 自动删除。

**LinkedHashMap 链表结构**

LinkedHashMap 新增了哪些属性，以达到了链表结构的：

```java
// 链表头
transient LinkedHashMap.Entry<K,V> head;
 
// 链表尾
transient LinkedHashMap.Entry<K,V> tail;
 
// 继承 Node，为数组的每个元素增加了 before 和 after 属性
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
 
// 控制两种访问模式的字段，默认 false
// true 按照访问顺序，会把经常访问的 key 放到队尾
// false 按照插入顺序提供访问
final boolean accessOrder;
```

LinkedHashMap 的数据结构很像是把 LinkedList 的每个元素换成了 HashMap 的 Node，像是两者的结合体，也正是因为增加了这些结构，从而能把 Map 的元素都串联起来，形成一个链表，而链表就可以保证顺序了，就可以维护元素插入进来的顺序。

<img src="https://img-blog.csdnimg.cn/1d25c9103ed741ceb5676e00bd5af3ad.png" alt="img" style="zoom:25%;" />

**如何按照顺序新增**

LinkedHashMap 初始化时，默认 accessOrder 为 false，就是会按照插入顺序提供访问，插入方法使用的是父类 HashMap 的 put 方法，不过覆写了 put 方法执行中调用的 newNode/newTreeNode 和 afterNodeAccess 方法。

newNode/newTreeNode 方法，控制新增节点追加到链表的尾部，这样每次新节点都追加到尾部，即可保证插入顺序了，我们以 newNode 源码为例：

```java
// 新增节点，并追加到链表的尾部
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    // 新增节点
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    // 追加到链表的尾部
    linkNodeLast(p);
    return p;
}
// link at the end of list
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    // 新增节点等于位节点
    tail = p;
    // last 为空，说明链表为空，首尾节点相等
    if (last == null)
        head = p;
    // 链表有数据，直接建立新增节点和上个尾节点之间的前后关系即可
    else {
        p.before = last;
        last.after = p;
    }
}
```

LinkedHashMap 通过新增头节点、尾节点，给每个节点增加 before、after 属性，每次新增时，都把节点追加到尾节点等手段，在新增的时候，就已经维护了按照插入顺序的链表结构了。

**按照顺序访问**

LinkedHashMap 只提供了单向访问，即按照插入的顺序从头到尾进行访问，不能像 LinkedList 那样可以双向访问。

我们主要通过迭代器进行访问，迭代器初始化的时候，默认从头节点开始访问，在迭代的过程中，不断访问当前节点的 after 节点即可。

```java
// 初始化时，默认从头节点开始访问
LinkedHashIterator() {
    // 头节点作为第一个访问的节点
    next = head;
    expectedModCount = modCount;
    current = null;
}
 
final LinkedHashMap.Entry<K,V> nextNode() {
    LinkedHashMap.Entry<K,V> e = next;
    if (modCount != expectedModCount)// 校验
        throw new ConcurrentModificationException();
    if (e == null)
        throw new NoSuchElementException();
    current = e;
    next = e.after; // 通过链表的 after 结构，找到下一个迭代的节点
    return e;
}
```

在新增节点时，我们就已经维护了元素之间的插入顺序了，所以迭代访问时非常简单，只需要不断的访问当前节点的下一个节点即可。

**访问最少删除策略**

这种策略也叫做 LRU（最近最少使用），大概的意思就是经常访问的元素会被追加到队尾，这样不经常访问的数据自然就靠近队头，然后我们可以通过设置删除策略，比如当 Map 元素个数大于多少时，把头节点删除。

**小结**

LinkedHashMap 提供了两个很有意思的功能：按照插入顺序访问和删除最少访问元素策略。