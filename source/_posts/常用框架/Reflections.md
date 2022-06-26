---
title: Reflections
categories: 
- 常用框架
---

**它能做什么？**

* 获取某个类型的全部子类

* 只要类型、构造器、方法，字段上带有特定注解，便能获取带有这个注解的全部信息（类型、构造器、方法，字段）

* 获取所有能匹配某个正则表达式的资源

* 获取所有带有特定签名的方法，包括参数，参数注解，返回类型

* 获取所有方法的名字

* 获取代码里所有字段、方法名、构造器的使用

**相关资料**

github地址 ： https://github.com/ronmamo/reflections

javadoc地址 : http://ronmamo.github.io/reflections/index.html?org/reflections/Reflections.html

```xml
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId>
    <version>0.9.11</version>
</dependency>
```

```java
@Test
    public void testReflections() {
        Reflections reflections = new Reflections("org.fhp.test.entity");
        Set<Class<? extends MyInterface>> classes = reflections.getSubTypesOf(MyInterface.class);
 
        for(Class clazz : classes) {
            //logger.info(clazz.getName());
            System.out.println("Found: " + clazz.getName());
        }
    }
```

假如想扫描整个工程的类，直接new一个不带参数的Reflections就好。

值得一提的是，这东西在扫描的时候，连依赖的jar包都不放过。

以Spring框架的BeanFactory为例：

```java
@Test
    public void testReflections() {
        Reflections reflections = new Reflections();
        Set<Class<? extends BeanFactory>> classes = reflections.getSubTypesOf(BeanFactory.class);
 
        for(Class clazz : classes) {
            //logger.info(clazz.getName());
            System.out.println("Found: " + clazz.getName());
        }
    }
```

另一个常用的场景是扫描包含指定注解的类。reflections对象中同样包含这一方法，代码如下：

```java
@Test
    public void testReflections() {
        Reflections reflections = new Reflections();
        Set<Class<?>> classes = reflections.getTypesAnnotatedWith(Service.class);
 
        for(Class clazz : classes) {
            //logger.info(clazz.getName());
            System.out.println("Found: " + clazz.getName());
        }
    }
```

