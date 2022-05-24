---
title: TreeSet
categories: 
- 编程语言
- Java基础
- 集合类
---

TreeSet是一个有序的集合，它的作用是提供有序的Set集合。

它继承了AbstractSet抽象类，实现了NavigableSet，Cloneable，Serializable接口。

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
```

TreeSet是基于TreeMap实现的，TreeSet的元素支持2种排序方式：自然排序或者根据提供的Comparator进行排序。

**TreeSet用法**

```java
public static void demoOne() {
        TreeSet<Person> ts = new TreeSet<>();
        ts.add(new Person("张三", 11));
        ts.add(new Person("李四", 12));
        ts.add(new Person("王五", 15));
        ts.add(new Person("赵六", 21));
        
        System.out.println(ts);
    }
```

执行结果：会抛出一个 异常：`java.lang.ClassCastException`

原因在于我们需要告诉TreeSet如何来进行比较元素，如果不指定，就会抛出这个异常

**如何解决：**

如何指定比较的规则，需要在自定义类(Person)中实现Comparable接口，并重写接口中的compareTo方法

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;
    ...
    public int compareTo(Person o) {
        return 0;                //当compareTo方法返回0的时候集合中只有一个元素
        return 1;                //当compareTo方法返回正数的时候集合会怎么存就怎么取
        return -1;                //当compareTo方法返回负数的时候集合会倒序存储
    }
}
```

**源码分析**

TreeSet如何保证元素不重复以及元素有序的，它是基于TreeMap实现的。

```java
private transient NavigableMap<E,Object> m; // 保证有序

// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object(); // 固定Value
```

看`add`和`remove`方法：

```java
public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }
    
public boolean remove(Object o) {
        return m.remove(o)==PRESENT;
    }
```

和HashSet的实现一样，也是利用了Map保存的Key-Value键值对的Key不会重复的特点。

**总结**

1. TreeSet是基于TreeMap实现的，支持自然排序和自定义排序，可以进行逆序输出；
2. TreeSet不允许null值；
3. TreeSet不是线程安全的，多线程环境下可以使用`SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...))`；