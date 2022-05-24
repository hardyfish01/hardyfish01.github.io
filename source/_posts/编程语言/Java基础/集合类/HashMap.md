---
title: HashMap
categories: 
- 编程语言
- Java基础
- 集合类
---

**整体架构**

HashMap 底层的数据结构主要是：数组 + 链表 + 红黑树。

其中当链表的长度大于等于 8 时，链表会转化成红黑树，当红黑树的大小小于等于 6 时，红黑树会转化成链表，整体的数据结构如下：

<img src="https://img-blog.csdnimg.cn/c57cb7c4478d484b87e5088cfef1517c.png" alt="img" style="zoom:25%;" />

**基本特点**

- 允许 null 值，不同于 HashTable ，是线程不安全的；
- load factor（影响因子） 默认值是 0.75， 是均衡了时间和空间损耗算出来的值，较高的值会减少空间开销（扩容减少，数组大小增长速度变慢），但增加了查找成本（hash 冲突增加，链表长度变长），不扩容的条件：`数组容量 > 需要的数组大小 /load factor`；
- 如果有很多数据需要储存到 HashMap 中，建议 HashMap 的容量一开始就设置成足够的大小，这样可以防止在其过程中不断的扩容，影响性能；
- HashMap 是非线程安全的，我们可以自己在外部加锁，或者通过 `Collections#synchronizedMap` 来实现线程安全，`Collections#synchronizedMap` 的实现是在每个方法上加上了 synchronized 锁；
- 在迭代过程中，如果 HashMap 的结构被修改，会快速失败。

**常见属性**

```java
 //初始容量为 16
 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
 
 //最大容量
 static final int MAXIMUM_CAPACITY = 1 << 30;
 
 //负载因子默认值
 static final float DEFAULT_LOAD_FACTOR = 0.75f;
 
 //桶上的链表长度大于等于8时，链表转化成红黑树
 static final int TREEIFY_THRESHOLD = 8;
 
 //桶上的红黑树大小小于等于6时，红黑树转化成链表
 static final int UNTREEIFY_THRESHOLD = 6;
 
 //当数组容量大于 64 时，链表才会转化成红黑树
 static final int MIN_TREEIFY_CAPACITY = 64;
 
 //记录迭代过程中 HashMap 结构是否发生变化，如果有变化，迭代时会 fail-fast
 transient int modCount;
 
 //HashMap 的实际大小，可能不准(因为当你拿到这个值的时候，可能又发生了变化)
 transient int size;
 
 //存放数据的数组
 transient Node<K,V>[] table;
 
 // 扩容的门槛，有两种情况
 // 如果初始化时，给定数组大小的话，通过 tableSizeFor 方法计算，数组大小永远接近于 2 的幂次方，比如你给定初始化大小 19，实际上初始化大小为 32，为 2 的 5 次方。
 // 如果是通过 resize 方法进行扩容，大小 = 数组容量 * 0.75
 int threshold;
 
 //链表的节点
 static class Node<K,V> implements Map.Entry<K,V> {
 
 //红黑树的节点
 static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
```

# 查找

**HashMap 的查找主要分为以下三步：**

- 根据 hash 算法定位数组的索引位置，equals 判断当前节点是否是我们需要寻找的 key，是的话直接返回，不是的话往下。
- 判断当前节点有无 next 节点，有的话判断是链表类型，还是红黑树类型。
- 分别走链表和红黑树不同类型的查找方法。

链表查找的关键代码是：

```java
// 采用自旋方式从链表中查找 key，e 初始为为链表的头节点
do {
    // 如果当前节点 hash 等于 key 的 hash，并且 equals 相等，当前节点就是我们要找的节点
    // 当 hash 冲突时，同一个 hash 值上是一个链表的时候，我们是通过 equals 方法来比较 key 是否相等的
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        return e;
    // 否则，把当前节点的下一个节点拿出来继续寻找
} while ((e = e.next) != null);
```

# 新增

**新增 key，value 大概的步骤如下：**

1. 空数组有无初始化，没有的话初始化；
2. 如果通过 key 的 hash 能够直接找到值，跳转到 6，否则到 3；
3. 如果 hash 冲突，两种解决方案：链表 or 红黑树；
4. 如果是链表，递归循环，把新元素追加到队尾；
5. 如果是红黑树，调用红黑树新增的方法；
6. 通过 2、4、5 将新元素追加成功，再根据 onlyIfAbsent 判断是否需要覆盖；
7. 判断是否需要扩容，需要扩容进行扩容，结束。

<img src="https://img-blog.csdnimg.cn/c62c30ef5f884542855a2ec81c270450.png" alt="img" style="zoom:25%;" />

```java
// 入参 hash：通过 hash 算法计算出来的值。
// 入参 onlyIfAbsent：false 表示即使 key 已经存在了，仍然会用新值覆盖原来的值，默认为 false
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // n 表示数组的长度，i 为数组索引下标，p 为 i 下标位置的 Node 值
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //如果数组为空，使用 resize 方法初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 如果当前索引位置是空的，直接生成新的节点在当前索引位置上
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 如果当前索引位置有值的处理方法，即我们常说的如何解决 hash 冲突
    else {
        // e 当前节点的临时变量
        Node<K,V> e; K k;
        // 如果 key 的 hash 和值都相等，直接把当前下标位置的 Node 值赋值给临时变量
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果是红黑树，使用红黑树的方式新增
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 是个链表，把新节点放到链表的尾端
        else {
            // 自旋
            for (int binCount = 0; ; ++binCount) {
                // e = p.next 表示从头开始，遍历链表
                // p.next == null 表明 p 是链表的尾节点
                if ((e = p.next) == null) {
                    // 把新节点放到链表的尾部 
                    p.next = newNode(hash, key, value, null);
                    // 当链表的长度大于等于 8 时，链表转红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }
                // 链表遍历过程中，发现有元素和新增的元素相等，结束循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                //更改循环的当前元素，使 p 在遍历过程中，一直往后移动。
                p = e;
            }
        }
        // 说明新节点的新增位置已经找到了
        if (e != null) {
            V oldValue = e.value;
            // 当 onlyIfAbsent 为 false 时，才会覆盖值 
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            // 返回老值
            return oldValue;
        }
    }
    // 记录 HashMap 的数据结构发生了变化
    ++modCount;
    //如果 HashMap 的实际大小大于扩容的门槛，开始扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

当链表长度大于等于 8 时，此时的链表就会转化成红黑树，转化的方法是：treeifyBin，此方法有一个判断，当链表长度大于等于 8，并且整个数组大小大于 64 时，才会转成红黑树，当数组大小小于 64 时，只会触发扩容，不会转化成红黑树。

**为什么是 8，这个答案在源码中注释有说，中文翻译过来大概的意思是：**

链表查询的时间复杂度是 `O(n)`，红黑树的查询复杂度是` O(log(n))`。

在链表数据不多的时候，使用链表进行遍历也比较快，只有当链表数据比较多的时候，才会转化成红黑树，但红黑树需要的占用空间是链表的 2 倍，考虑到转化时间和空间损耗，所以我们需要定义出转化的边界值。

在考虑设计 8 这个值的时候，我们参考了泊松分布概率函数，由泊松分布中得出结论，链表各个长度的命中概率为：

```
* 0:    0.60653066
* 1:    0.30326533
* 2:    0.07581633
* 3:    0.01263606
* 4:    0.00157952
* 5:    0.00015795
* 6:    0.00001316
* 7:    0.00000094
* 8:    0.00000006
```

意思是，当链表的长度是 8 的时候，出现的概率是 0.00000006，不到千万分之一，所以说正常情况下，链表的长度不可能到达 8 ，而一旦到达 8 时，肯定是 Hash 算法出了问题，所以在这种情况下，为了让 HashMap 仍然有较高的查询性能，所以让链表转化成红黑树，我们正常写代码，使用 HashMap 时，几乎不会碰到链表转化成红黑树的情况，毕竟概念只有千万分之一。

# 扩容

**什么时候进行扩容**

1. 当HashMap中的元素个数超过`数组大小*loadFactor`时，就会进行数组扩容，loadFactor的默认值为0.75。
2. 数组长度小于64，且某个桶中的元素大于8。

**为什么扩容是2的次幂**

容量是2的n次幂，可以使得添加的元素均匀分布在HashMap中的数组上，减少Hash碰撞，避免形成链表的结构，使得查询效率降低。

根据数学规律，对n取模，就是和n-1进行与运算，与运算的效率远远高于求模运算。

当数组长度是2的幂次时，hash值对数组长度取模的效果和`hash&(n-1)`是一样的。

**扩容步骤**

当这个数组满了以后，就会自动扩容，变成一个更大的数组，可以在里面放更多的元素。

1. 在resize()方法中，定义了`oldCap`参数，记录了原table的长度，定义了`newCap`参数，记录新table长度，newCap是oldCap长度的2倍。
2. 循环原table，把原`table`中的每个链表中的每个元素放入新table。

假设table原长度是16，扩容后长度32，那么一个`hash`值在扩容前后的table下标是这么计算的：

- 判断条件`(e.hash & oldCap) == 0` 计算出来的值，如果等于0，就放入低位节点，低位节点在新数组的位置跟原数组一样。
- 等于1，就放入高位节点，高位放在新数组的位置等于原来位置在加上原数组的长度。

```java
final Node<K,V>[] resize() {
    // 扩容前的数组
    Node<K,V>[] oldTab = table;
    // 扩容前的数组的大小和阈值
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    // 预定义新数组的大小和阈值
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩容了
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 扩大容量为当前容量的两倍，但不能超过 MAXIMUM_CAPACITY
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 当前数组没有数据，使用初始化的值
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        // 如果初始化的值为 0，则使用默认的初始化容量
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 如果新的容量等于 0
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr; 
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    // 开始扩容，将新的容量赋值给 table
    table = newTab;
    // 原数据不为空，将原数据复制到新 table 中
    if (oldTab != null) {
        // 根据容量循环数组，复制非空元素到新 table
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 如果链表只有一个，则进行直接赋值
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 红黑树相关的操作
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    // 链表复制，JDK 1.8 扩容优化部分
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引 + oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 将原索引放到哈希桶中
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 将原索引 + oldCap 放到哈希桶中
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

# 常见面试题

**HashMap和HashTable的区别**

1. Hashtable 是不允许键或值为 null 的，HashMap 的键值则都可以为 null，在HashMap 中不能用get()方法来判断HashMap中是否存在某个键，而应该用`containsKey()`方法来判断
2. Hashtable 继承了 Dictionary类，而 HashMap 继承的是 AbstractMap 类，`Dictionary`的子类同时也实现了`Map`接口
3. HashMap 的初始容量为：16，Hashtable 初始容量为：11，两者的负载因子默认都是：0.75，当现有容量大于`总容量 * 负载因子`时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 + 1，`int newCapacity = (oldCapacity << 1) + 1;`
4. HashTable 是直接使用key的hashCode(`key.hashCode()`)作为hash值，不像 HashMap 内部使用 扰动函数对key的hashCode进行扰动后作为hash值
5. `HashTable`取哈希桶下标是直接用模运算`%`（因为其默认容量也不是2的n次方。所以也无法用位运算替代模运算）
6. Hashtable是线程安全的，方法是Synchronized的

**加载因子为什么是0.75**

加载因子也叫扩容因子或负载因子，用来判断什么时候进行扩容的，假如加载因子是 0.5，`HashMap` 的初始化容量是 16，那么当 HashMap 中有 `16*0.5=8` 个元素时，HashMap 就会进行扩容。

这其实是出于容量和性能之间平衡的结果：

- 当加载因子设置比较大的时候，扩容的门槛就被提高了，扩容发生的频率比较低，占用的空间会比较小，但此时发生 Hash 冲突的几率就会提升，因此需要更复杂的数据结构来存储元素，这样对元素的操作时间就会增加，运行效率也会因此降低；
- 而当加载因子值比较小的时候，扩容的门槛会比较低，因此会占用更多的空间，此时元素的存储就比较稀疏，发生哈希冲突的可能性就比较小，因此操作性能会比较高。

所以综合了以上情况就取了一个 0.5 到 1.0 的平均数 0.75 作为加载因子。