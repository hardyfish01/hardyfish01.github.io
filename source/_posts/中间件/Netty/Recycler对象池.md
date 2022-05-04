---
title: Recycler对象池
categories: 
- 中间件
- Netty
---

对象池与内存池的都是为了提高 Netty 的并发处理能力，我们知道 Java 中频繁地创建和销毁对象的开销是很大的，所以很多人会将一些通用对象缓存起来，当需要某个对象时，优先从对象池中获取对象实例。

通过重用对象，不仅避免频繁地创建和销毁所带来的性能损耗，而且对 JVM GC 是友好的，这就是对象池的作用。

Recycler 是 Netty 提供的自定义实现的轻量级对象回收站，借助 Recycler 可以完成对象的获取和回收

```java
public class RecyclerDemo {
    private static final Recycler<User> RECYCLER = new Recycler<User>() {
        @Override
        protected User newObject(Handle<User> handle) {
            return new User(handle);
        }
    };
    static class User{
        private final Recycler.Handle<User> handle;
        public User(Recycler.Handle<User> handle){
            this.handle=handle;
        }
        public void recycle(){
            handle.recycle(this);
        }
    }
    public static void main(String[] args){
        User user1 = RECYCLER.get();
        user1.recycle();
        User user2 = RECYCLER.get();
        user2.recycle();
        System.out.println(user1==user2);
    }
}
```

首先定义了一个Recycler的成员变量RECYCLER，在匿名内部类中重写了newObject方法，也就是创建对象的方法，该方法就是用户自定义的，这里newObject返回的`new User(handle)`，代表当回收站没有此类对象的时候，可以通过这种方式创建对象

**成员变量RECYCLER, 可以用来对此类对象的回收和再利用**

定义了一个一个静态内部类User，User中有个成员变量handle，在构造方法中为其赋值，handle的作用就是用于对象回收的，并且定义了一个方法recycle，方法体中通过`handle.recycle(this)`这种方式将自身对象进行回收，通过这步操作就可以将对象回收到Recycler中，在main方法中，通过RECYCLER的get方法获取一个user，然后进行回收，再通过get方法将回收站的对象取出，再次进行回收，最后判断两次取出的对象是否为一个对象， 最后结果输出为true