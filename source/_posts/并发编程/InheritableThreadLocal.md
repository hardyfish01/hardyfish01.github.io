---
title: InheritableThreadLocal
categories: 
- 并发编程
---

InheritableThreadLocal主要用于子线程创建时，需要自动继承父线程的ThreadLocal变量，方便必要信息的进一步传递。

**我们先来看看例子：**

```java
public class InheritableThreadLocalT {
    public static final ThreadLocal<String> threadLocal = new ThreadLocal<>();
 
    public static void main(String[] args) {
        threadLocal.set("hello");
        System.out.println(threadLocal.get());
 
        new Thread() {
            @Override
            public void run() {
                System.out.println(threadLocal.get());
            }
        }.start();
    }
}
```

![](https://img-blog.csdnimg.cn/a39aceb994c84005ad53a274b1901443.png)

再来看看使用另外的：

```java
public class InheritableThreadLocalT {
    public static final InheritableThreadLocal<String> threadLocal = new InheritableThreadLocal<>();
 
    public static void main(String[] args) {
        threadLocal.set("hello");
        System.out.println(threadLocal.get());
 
        new Thread() {
            @Override
            public void run() {
                System.out.println(threadLocal.get());
                new Thread(){
                    @Override
                    public void run() {
                        System.out.println(threadLocal.get());
                    }
                }.start();
            }
        }.start();
    }
}
```

![](https://img-blog.csdnimg.cn/09b991200598444b9d758a82a1f0289d.png)

# 原理

`InheritableThreadLocal`继承于`ThreadLocal`，并重写了`ThreadLocal`中的三个方法。

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    protected T childValue(T parentValue) {
        return parentValue;
    }

    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

**线程的创建过程**

跟踪线程的创建`new Thread()`方法。

> 进入初始化方法。

```java
public Thread() {
  init(null, null, "Thread-" + nextThreadNum(), 0);
}
```

> 调用`init`方法。

重载对应的`init`方法，然后使用当前线程(父线程)获取到对应的`ThreadLocalMap`，然后传递给`inheritableThreadLocals`变量。

```java
private void init(ThreadGroup g, Runnable target, String name,long stackSize) {
  init(g, target, name, stackSize, null, true);
}
```

通过重载方法之后，最后实际调用的处理方法。`parent`线程为创建子线程的当前线程，也就是父线程。

`inheritThreadLocals`的默认值是`true`，且父线程的`inheritableThreadLocal`对象不为空。 

则创建当前线程的`inheritableThreadLocals`对象。

```java
private void init() {
  this.name = name;
  Thread parent = currentThread();
  	
  //   ...
  if (inheritThreadLocals && parent.inheritableThreadLocals != null)
    this.inheritableThreadLocals =
    ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
}
```

> 进入创建方法`createInheritedMap`方法。

以父线程的`inheritableThreadLocals`为实例创建子线程的`inheritableThreadLocals`对象实现上比较简单，将父线程的`inheritableThreadLocals`循环拷贝给子线程。

以父线程的`inheritableThreadLocals`为实例创建一个`ThreadLocalMap`对象。

```java
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
  return new ThreadLocalMap(parentMap);
}
```

