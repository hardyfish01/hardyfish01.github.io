---
title: LinkedList
categories: 
- 编程语言
- Java基础
- 集合类
---

[ArrayList和LinkedList使用不当，性能差距会如此之大！](https://mp.weixin.qq.com/s/IuXnprZdz2Au6yjAgJ1hcg)

**LinkedList简介**

1. `LinkedList`是一个继承于`AbstractSequentialList`的双向链表。它也可以被当做堆栈、队列或双端队列进行使用。
2. `LinkedList`实现`List`接口，能让它进行队列操作。
3. `LinkedList`实现`Deque`接口，即能将`LinkedList`当做双端队列使用。
4. `LinkedList`实现`Cloneable`，即覆盖了函数`clone()`，能被克隆。
5. `LinkedList`实现了`java.io.Serializable`接口，这意味着`LinkedList`支持序列化，能通过序列化去传输。
6. `LinkedList`中的操作不是线程安全的。

**整体架构**

LinkedList 底层数据结构是一个双向链表，整体结构如下图所示：

<img src="https://img-blog.csdnimg.cn/fbba702262574c7cb013087404b08151.png" alt="img" style="zoom:25%;" />

上图代表了一个双向链表结构，链表中的每个节点都可以向前或者向后追溯：

- 链表每个节点我们叫做 Node，Node 有 prev 属性，代表前一个节点的位置，next 属性，代表后一个节点的位置；
- first 是双向链表的头节点，它的前一个节点是 null。
- last 是双向链表的尾节点，它的后一个节点是 null；
- 当链表中没有数据时，first 和 last 是同一个节点，前后指向都是 null；
- 因为是个双向链表，只要机器内存足够强大，是没有大小限制的。

链表中的元素叫做 Node：

```java
private static class Node<E> {
    E item;// 节点值
    Node<E> next; // 指向的下一个节点
    Node<E> prev; // 指向的前一个节点
 
    // 初始化参数顺序分别是：前一个节点、本身节点值、后一个节点
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

**追加（新增）**

追加节点时，我们可以选择追加到链表头部，还是追加到链表尾部，add 方法默认是从尾部开始追加，addFirst 方法是从头部开始追加。

**从尾部追加（add）**

```java
// 从尾部开始追加节点
void linkLast(E e) {
    // 把尾节点数据暂存
    final Node<E> l = last;
    // 新建新的节点，初始化入参含义：
    // l 是新节点的前一个节点，当前值是尾节点值
    // e 表示当前新增节点，当前新增节点后一个节点是 null
    final Node<E> newNode = new Node<>(l, e, null);
    // 新建节点追加到尾部
    last = newNode;
    //如果链表为空（l 是尾节点，尾节点为空，链表即空），头部和尾部是同一个节点，都是新建的节点
    if (l == null)
        first = newNode;![图片描述](//img.mukewang.com/5d5fc69600013e4803600240.gif)
    //否则把前尾节点的下一个节点，指向当前尾节点。
    else
        l.next = newNode;
    //大小和版本更改
    size++;
    modCount++;
}
```

**节点删除**

节点删除的方式和追加类似，我们可以选择从头部删除，也可以选择从尾部删除，删除操作会把节点的值，前后指向节点都置为 null，帮助 GC 进行回收。

```java
//从头删除节点 f 是链表头节点
private E unlinkFirst(Node<E> f) {
    // 拿出头节点的值，作为方法的返回值
    final E element = f.item;
    // 拿出头节点的下一个节点
    final Node<E> next = f.next;
    //帮助 GC 回收头节点
    f.item = null;
    f.next = null;
    // 头节点的下一个节点成为头节点
    first = next;
    //如果 next 为空，表明链表为空
    if (next == null)
        last = null;
    //链表不为空，头节点的前一个节点指向 null
    else
        next.prev = null;
    //修改链表大小和版本
    size--;
    modCount++;
    return element;
}
```

**节点查询**

链表查询某一个节点是比较慢的，需要挨个循环查找才行。

```java
// 根据链表索引位置查询节点
Node<E> node(int index) {
    // 如果 index 处于队列的前半部分，从头开始找，size >> 1 是 size 除以 2 的意思。
    if (index < (size >> 1)) {
        Node<E> x = first;
        // 直到 for 循环到 index 的前一个 node 停止
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {// 如果 index 处于队列的后半部分，从尾开始找
        Node<E> x = last;
        // 直到 for 循环到 index 的后一个 node 停止
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

**总结**

LinkedList 适用于要求有顺序、并且会按照顺序进行迭代的场景，主要是依赖于底层的链表结构。