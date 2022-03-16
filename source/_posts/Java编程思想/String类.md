---
title: String类
categories: 
- Java编程思想
---

HashMap 的 key 建议使用不可变类，比如说 String 这种不可变类。

> 这里说的不可变指的是类值一旦被初始化，就不能再被改变了，如果被修改，将会是新的类。

以主流的 JDK 版本 1.8 来说，String 内部实际存储结构为 `char` 数组，源码如下：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}    
```

注意两点：

1. String 被 final 修饰，说明 String 类绝不可能被继承了，也就是说任何对 String 的操作方法，都不会被继承覆写；
2. String 中保存数据的是一个 char 的数组 value。我们发现 value 也是被 final 修饰的，也就是说 value 一旦被赋值，内存地址是绝对无法修改的，而且 value 的权限是 private 的，外部绝对访问不到，String 也没有开放出可以对 value 进行赋值的方法，所以说 value 一旦产生，内存地址就根本无法被修改。

**String不可变的好处**

> 使用字符串常量池

> 用作 HashMap 的 key

String 不可变的第二个好处就是它可以很方便地用作 **HashMap （或者 HashSet） 的 key**。

* 通常建议把**不可变对象作为 HashMap的 key**，比如 String 就很合适作为 HashMap 的 key。

> 缓存 HashCode

在 Java 中经常会用到字符串的 HashCode，在 String 类中有一个 hash 属性，代码如下：

```java
/** Cache the hash code for the String */
private int hash;
```

这是一个成员变量，保存的是 String 对象的 HashCode。

因为 String 是不可变的，所以对象一旦被创建之后，HashCode 的值也就不可能变化了，我们就可以把 HashCode 缓存起来。

这样的话，以后每次想要用到 HashCode 的时候，不需要重新计算，直接返回缓存过的 hash 的值就可以了，因为它不会变，这样可以提高效率，所以这就使得字符串非常适合用作 HashMap 的 key。

而对于其他的不具备不变性的普通类的对象而言，如果想要去获取它的 HashCode ，就必须每次都重新算一遍，相比之下，效率就低了。

> 线程安全

具备**不变性的对象一定是线程安全的**，我们不需要对其采取任何额外的措施，就可以天然保证线程安全。

由于 String 是不可变的，所以它就可以非常安全地被多个线程所共享，这对于多线程编程而言非常重要，避免了很多不必要的同步操作。

**编译器还会对 String 字符串做一些优化，例如以下代码：**

```java
String s1 = "Ja" + "va";
String s2 = "Java";
System.out.println(s1 == s2);
```

虽然 s1 拼接了多个字符串，但对比的结果却是 true，我们使用反编译工具，看到的结果：

从编译代码 可以看出，代码 "Ja"+"va" 被直接编译成了 "Java" ，因此 `s1==s2` 的结果才是 true，这就是编译器对字符串优化的结果。