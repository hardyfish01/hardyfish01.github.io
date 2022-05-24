---
title: ArrayList
categories: 
- 编程语言
- Java基础
- 集合类
---

[从源码角度解析ArrayList.subList的几个坑](https://mp.weixin.qq.com/s/DNXAP3PGHE1zX_ax0TyA3Q)

[ArrayList和LinkedList使用不当，性能差距会如此之大！](https://mp.weixin.qq.com/s/IuXnprZdz2Au6yjAgJ1hcg)

**ArrayList 整体架构是一个数组结构，比较简单，如下图：**

<img src="https://img-blog.csdnimg.cn/6d8ff58aa3764d31be68f7fe4bace16c.png" alt="img" style="zoom:25%;" />

图中展示是长度为 10 的数组，从 1 开始计数，index 表示数组的下标，从 0 开始计数，elementData 表示数组本身。

**ArrayList 的继承关系**

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

**几个全局变量：**

```java
/**
 * ArrayList 默认的数组容量
 */
 private static final int DEFAULT_CAPACITY = 10;

/**
 * 这是一个共享的空的数组实例，当使用 ArrayList(0) 或者 ArrayList(Collection<? extends E> c) 
 * 并且 c.size() = 0 的时候讲 elementData 数组讲指向这个实例对象。
 */
 private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * 另一个共享空数组实例，再第一次 add 元素的时候将使用它来判断数组大小是否设置为 DEFAULT_CAPACITY
 */
 private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 真正装载集合元素的底层数组 
 * 至于 transient 关键字这里简单说一句，被它修饰的成员变量无法被 Serializable 序列化 
 */
transient Object[] elementData; // non-private to simplify nested class access
```

**无参构造方法**

```java
/**
 * 构造一个初始容量为10的空列表。
 */
public ArrayList() {
   this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

**指定初始容量的构造方法**

```java
/**
 * 构造一个具有指定初始容量的空列表。
 * @param  初始容量 
 * @throws 如果参数小于 0 将会抛出 IllegalArgumentException  参数不合法异常
 */
public ArrayList(int initialCapacity) {
   if (initialCapacity > 0) {
       this.elementData = new Object[initialCapacity];
   } else if (initialCapacity == 0) {
       this.elementData = EMPTY_ELEMENTDATA;
   } else {
       throw new IllegalArgumentException("Illegal Capacity: "+
                                          initialCapacity);
   }
}
```

**新增和扩容实现**

新增就是往数组中添加元素，主要分成两步：

- 判断是否需要扩容，如果需要执行扩容操作；
- 直接赋值。

```java
public boolean add(E e) {
  //确保数组大小是否足够，不够执行扩容，size 为当前数组的大小
  ensureCapacityInternal(size + 1);  // Increments modCount!!
  //直接赋值，线程不安全的
  elementData[size++] = e;
  return true;
}
```

**线程安全**

只有当 ArrayList 作为共享变量时，才会有线程安全问题，当 ArrayList 是方法内的局部变量时，是没有线程安全的问题的。

ArrayList 有线程安全问题的本质，是因为 ArrayList 自身的 elementData、size、modConut 在进行各种操作时，都没有加锁，而且这些变量的类型并非是可见（volatile）的，所以如果多个线程对这些变量进行操作时，可能会有值被覆盖的情况。

类注释中推荐我们使用 `Collections#synchronizedList` 来保证线程安全，SynchronizedList 是通过在每个方法上面加上锁来实现，虽然实现了线程安全，但是性能大大降低，具体实现源码：

```java
public boolean add(E e) {
    synchronized (mutex) {// synchronized 是一种轻量锁，mutex 表示一个当前 SynchronizedList
        return c.add(e);
    }
}
```

**面试题**

> ArrayList无参数构造器构造，现在 add 一个值进去，此时数组的大小是多少，下一次扩容前最大可用大小是多少？

此处数组的大小是 1，下一次扩容前最大可用大小是 10，因为 ArrayList 第一次扩容时，是有默认值的，默认值是 10，在第一次 add 一个值进去时，数组的可用大小被扩容到 10 了。

> 如果我连续往 list 里面新增值，增加到第 11 个的时候，数组的大小是多少？

当增加到 11 的时候，此时我们希望数组的大小为 11，但实际上数组的最大容量只有 10，不够了就需要扩容，扩容的公式是：`oldCapacity + (oldCapacity>> 1)`，oldCapacity 表示数组现有大小，目前场景计算公式是：`10 + 10 ／2 = 15`，然后我们发现 15 已经够用了，所以数组的大小会被扩容到 15。

> 数组初始化，被加入一个值后，如果使用 addAll 方法，一下子加入 15 个值，那么最终数组的大小是多少？

现在需要一下子加入 15 个值，那我们期望数组的大小值就是 16，此时数组最大可用大小只有 10，明显不够，需要扩容，扩容后的大小是：`10 + 10 ／2 = 15`，这时候发现扩容后的大小仍然不到我们期望的值 16，这时候源码中有一种策略如下：

```java
// newCapacity 本次扩容的大小，minCapacity 我们期望的数组最小大小
// 如果扩容后的值 < 我们的期望值，我们的期望值就等于本次扩容的大小
if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;
```

所以最终数组扩容后的大小为 16。

> 现在有一个很大的数组需要拷贝，原数组大小是 5k，请问如何快速拷贝？

因为原数组比较大，如果新建新数组的时候，不指定数组大小的话，就会频繁扩容，频繁扩容就会有大量拷贝的工作，造成拷贝的性能低下，所以新建数组时，指定新数组的大小为 5k。

> 为什么说扩容会消耗性能？

扩容底层使用的是 `System.arraycopy` 方法，会把原数组的数据全部拷贝到新数组上，所以性能消耗比较严重。

> ArrayList 数组，我们通过增强 for 循环进行删除，可以么？

不可以，会报错。因为增强 for 循环过程其实调用的就是迭代器的 next () 方法，当你调用 `list#remove()` 方法进行删除时，modCount 的值会 +1，而这时候迭代器中的 expectedModCount 的值却没有变，导致在迭代器下次执行 next () 方法时，`expectedModCount != modCount` 就会报 ConcurrentModificationException 的错误。

> 如果删除时使用` Iterator.remove()`方法可以删除么，为什么？

可以的，因为`Iterator.remove()`方法在执行的过程中，会把最新的 modCount 赋值给 expectedModCount，这样在下次循环过程中，modCount 和 expectedModCount 两者就会相等。