---
title: TreeMap
categories: 
- 编程语言
- Java基础
- 集合类
---

**日常工作中排序的两种方式：**

```java
public class TreeMapDemo {
 
  @Data
  // DTO 为我们排序的对象
  class DTO implements Comparable<DTO> {
    private Integer id;
    public DTO(Integer id) {
      this.id = id;
    }
 
    @Override
    public int compareTo(DTO o) {
      //默认从小到大排序
      return id - o.getId();
    }
  }
 
  @Test
  public void testTwoComparable() {
    // 第一种排序，从小到大排序，实现 Comparable 的 compareTo 方法进行排序
    List<DTO> list = new ArrayList<>();
    for (int i = 5; i > 0; i--) {
      list.add(new DTO(i));
    }
    Collections.sort(list);
    log.info(JSON.toJSONString(list));
 
    // 第二种排序，从大到小排序，利用外部排序器 Comparator 进行排序
    Comparator comparator = (Comparator<DTO>) (o1, o2) -> o2.getId() - o1.getId();
    List<DTO> list2 = new ArrayList<>();
    for (int i = 5; i > 0; i--) {
      list2.add(new DTO(i));
    }
    Collections.sort(list,comparator);
    log.info(JSON.toJSONString(list2));
  }
}
```

**TreeMap概述**

- TreeMap继承了NavigableMap接口，NavigableMap接口继承了SortedMap接口；
- TreeMap实现了Cloneable接口，可被克隆，实现了Serializable接口，可序列化；
- TreeMap因为是通过红黑树实现，红黑树结构天然支持排序，默认情况下通过Key值的自然顺序进行排序；

**TreeMap 整体架构**

TreeMap 底层的数据结构就是红黑树，和 HashMap 的红黑树结构一样。

不同的是，TreeMap 利用了红黑树左节点小，右节点大的性质，根据 key 进行排序，使每个元素能够插入到红黑树大小适当的位置，维护了 key 的大小关系，适用于 key 需要排序的场景。

因为底层使用的是平衡红黑树的结构，所以 containsKey、get、put、remove 等方法的时间复杂度都是 log(n)。

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

1. 判断红黑树的节点是否为空，为空的话，新增的节点直接作为根节点。
2. 根据红黑树左小右大的特性，进行判断，找到应该新增节点的父节点。
3. 在父节点的左边或右边插入新增节点。
4. 着色旋转，达到平衡，结束。

<img src="https://img-blog.csdnimg.cn/8e62437a1b95438c81a00580e26edf7c.png" alt="img" style="zoom:25%;" />

**小结**

TreeMap 是通过 compare 来比较 key 的大小，然后利用红黑树左小右大的特性，为每个 key 找到自己的位置，从而维护了 key 的大小排序顺序。