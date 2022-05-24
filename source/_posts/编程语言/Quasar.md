---
title: Quasar
categories: 
- 编程语言
---

**协程是什么？**

我们知道，线程在阻塞状态和可运行状态的切换，以及线程间的上下文切换都会造成性能的损耗。

为了解决这些问题，引入协程`coroutine`这一概念，就像在一个进程中允许存在多个线程，在一个线程中，也可以存在多个协程。

<img src="https://img-blog.csdnimg.cn/16a952cabe7941198e5914c17e89e898.png" alt="img" style="zoom:25%;" />

**使用协程究竟有什么好处呢？**

> 执行效率高。线程的切换由操作系统内核执行，消耗资源较多。

而协程由程序控制，在用户态执行，不需要从用户态切换到内核态，我们也可以理解为，协程是一种进程自身来调度任务的调度模式，因此协程间的切换开销远小于线程切换。

> 节省资源。因为协程在本质上是通过分时复用了一个单线程，因此能够节省一定的资源。

类似于线程的五种状态切换，协程间也存在状态的切换。

虽然在Java官方的jdk中不能直接使用协程，但是，有其他的开源框架借助动态修改字节码的方式实现了协程。

**Quasar使用**

Quasar是一个开源的Java协程框架，通过利用`Java instrument`技术对字节码进行修改。

使方法挂起前后可以保存和恢复jvm栈帧，方法内部已执行到的字节码位置也通过增加状态机的方式记录，在下次恢复执行可直接跳转至最新位置。

网站：https://github.com/puniverse/quasar

引入pom文件：

```xml
<dependency>
    <groupId>co.paralleluniverse</groupId>
    <artifactId>quasar-core</artifactId>
    <version>0.7.10</version>
</dependency>
```

这里我们要使用的是Quasar中的Fiber，它可以被翻译为协程或纤程，创建Fiber的类型主要可分为下面两类：

```java
public Fiber(String name, FiberScheduler scheduler, int stackSize, SuspendableRunnable target);
public Fiber(String name, FiberScheduler scheduler, int stackSize, SuspendableCallable<V> target);
```

在Fiber中可以运行无返回值的SuspendableRunnable或有返回值的SuspendableCallable，看这个名字也知道区别就是java中的Runnable和Callable的区别了。

其余参数都可以省略，name为协程的名称，scheduler是调度器，默认使用FiberForkJoinScheduler，stackSize指定用于保存fiber调用栈信息的stack大小。

在下面的代码中，使用了`Fiber.sleep()`方法进行协程的休眠，和`Thread.sleep()`非常类似。

```java
public static void main(String[] args) throws InterruptedException {
    CountDownLatch countDownLatch=new CountDownLatch(10000);
    long start = System.currentTimeMillis();

    for (int i = 0; i < 10000; i++) {
        new Fiber<>(new SuspendableRunnable(){
            @Override
            public Integer run() throws SuspendExecution, InterruptedException {
                Fiber.sleep(1000);
                countDownLatch.countDown();
            }
        }).start();
    }

    countDownLatch.await();
    long end = System.currentTimeMillis();
    System.out.println("Fiber use:"+(end-start)+" ms");
}
```

**原理与应用**

在编译时框架会对代码进行扫描，如果方法带有`@Suspendable`注解，或抛出了`SuspendExecution`，或在配置文件`META-INF/suspendables`中指定该方法，那么Quasar就会修改生成的字节码，在`park`挂起方法的前后，插入一些字节码。

这些字节码会记录此时协程的执行状态，例如相关的局部变量与操作数栈，然后通过抛出异常的方式将cpu的控制权从当前协程交回到控制器，此时控制器可以再调度另外一个协程运行，并通过之前插入的那些字节码恢复当前协程的执行状态，使程序能继续正常执行。

看一下前面例子中的`SuspendableRunnable`和`SuspendableCallable`，它们的`run`方法上都抛出了`SuspendExecution`，其实这并不是一个真正的异常，仅作为识别挂起方法的声明，在实际运行中不会抛出。

当我们创建了一个`Fiber`，并在其中调用了其他方法时，如果想要Quasar的调度器能够介入，那么必须在使用时层层抛出这个异常或添加注解。