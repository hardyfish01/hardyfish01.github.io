---
title: Integer类
categories: 
- 编程语言
- Java基础
---

````java
# 一：自动装箱
Integer s1 = 2;
Integer s2 = 2;
System.out.println(s1 == s2);
# 答案为true

# 二：
Integer s1 = new Integer(2);
Integer s2 = new Integer(2);
System.out.println(s1 == s2);
# 答案为false
````

`new Integer`的比较是false，而自动装箱的比较却是true。

自动装箱等效于`Integer.valueOf()`，在valueOf()的方法中会缓存-127到128的Integer对象。

在valueOf()方法中，如果值在-127-128之间，都会直接返回缓存中的该对象而不会重新生成对象。

```java
   public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

而`new Integer()`的话并没有进行缓存，而且直接创建对象放入堆中，然后令栈中的2个变量引用它们，那么比较这两个变量也就是比较2个对象的内存地址当然会返回false。

除了Integer之外，其他包装类也有这种缓存机制：

* ByteCache：缓存Byte对象

* ShortChche：缓存Short对象

* LongChche：缓存Long对象

* CharacterChche：缓存Character对象

Byte，Short，Long的缓存范围都是-128-127，Character的缓存范围是0-127。