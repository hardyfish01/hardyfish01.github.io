---
title: Executor组件
categories: 
- Tomcat架构解析
---

为了提高处理能力和并发度，Web容器一般会把处理请求的工作放到线程池里来执行，Tomcat扩展了原生的Java线程池，来满足Web容器高并发的需求。

**通常我们可以将执行的任务分为两类：**

- cpu 密集型任务
- io 密集型任务

cpu 密集型任务，需要线程长时间进行的复杂的运算，这种类型的任务需要少创建线程，过多的线程将会频繁引起上文切换，降低任务处理处理速度

io 密集型任务，由于线程并不是一直在运行，可能大部分时间在等待 IO 读取/写入数据，增加线程数量可以提高并发度，尽可能多处理任务

* JDK线程池这种策略显然比较适合处理 cpu 密集型任务，但是对于 io 密集型任务，如数据库查询，rpc 请求调用等，就不是很友好了

由于 Tomcat/Jetty 需要处理大量客户端请求任务，如果采用原生线程池，一旦接受请求数量大于线程池核心线程数，这些请求就会被放入到队列中，等待核心线程处理，这样做显然降低这些请求总体处理速度，所以两者都没采用 JDK 原生线程池。

* 跟FixedThreadPool/CachedThreadPool一样，Tomcat的线程池也是一个定制版的ThreadPoolExecutor。

**定制版的ThreadPoolExecutor**

通过比较FixedThreadPool和CachedThreadPool，我们发现它们传给ThreadPoolExecutor的参数有两个关键点：

- 是否限制线程个数。
- 是否限制队列长度。

对于Tomcat来说，这两个资源都需要限制，也就是说要对高并发进行控制，否则CPU和内存有资源耗尽的风险。

因此Tomcat传入的参数是这样的：

```java
//定制版的任务队列
taskqueue = new TaskQueue(maxQueueSize);

//定制版的线程工厂
TaskThreadFactory tf = new TaskThreadFactory(namePrefix,daemon,getThreadPriority());

//定制版的线程池
executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), maxIdleTime, TimeUnit.MILLISECONDS,taskqueue, tf);
```

你可以看到其中的两个关键点：

- Tomcat有自己的定制版任务队列和线程工厂，并且可以限制任务队列的长度，它的最大长度是maxQueueSize。
- Tomcat对线程数也有限制，设置了核心线程数（minSpareThreads）和最大线程池数（maxThreads）。

除了资源限制以外，Tomcat线程池还定制自己的任务处理流程。

Tomcat线程池扩展了原生的ThreadPoolExecutor，通过重写execute方法实现了自己的任务处理逻辑：

1. 前corePoolSize个任务时，来一个任务就创建一个新线程。
2. 再来任务的话，就把任务添加到任务队列里让所有的线程去抢，如果队列满了就创建临时线程。
3. 如果总线程数达到maximumPoolSize，**则继续尝试把任务添加到任务队列中去。**
4. **如果缓冲队列也满了，插入失败，执行拒绝策略。**

观察Tomcat线程池和Java原生线程池的区别，其实就是在第3步，Tomcat在线程总数达到最大数时，不是立即执行拒绝策略，而是再尝试向任务队列添加任务，添加失败后再执行拒绝策略。

**定制版的任务队列**

只有当前线程数大于核心线程数、小于最大线程数，并且已提交的任务个数大于当前线程数时，也就是说线程不够用了，但是线程数又没达到极限，才会去创建新的线程。

这就是为什么Tomcat需要维护已提交任务数这个变量，它的目的就是在任务队列的长度无限制的情况下，让线程池有机会创建新的线程。

* 默认情况下Tomcat的任务队列是没有限制的，你可以通过设置maxQueueSize参数来限制任务队列的长度。

**总结**

池化的目的是为了避免频繁地创建和销毁对象，减少对系统资源的消耗。

* Java提供了默认的线程池实现，我们也可以扩展Java原生的线程池来实现定制自己的线程池，Tomcat就是这么做的。
* Tomcat扩展了Java线程池的核心类ThreadPoolExecutor，并重写了它的execute方法，定制了自己的任务处理流程。
* 同时Tomcat还实现了定制版的任务队列，重写了offer方法，使得在任务队列长度无限制的情况下，线程池仍然有机会创建新的线程。