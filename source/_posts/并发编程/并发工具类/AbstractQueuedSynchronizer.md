---
title: AbstractQueuedSynchronizer
categories: 
- 并发编程
- 并发工具类
---

`AbstractQueuedSynchronizer`抽象同步队列简称`AQS`，它是实现同步器的基础组件，如常用的`ReentrantLock、Semaphore、CountDownLatch`等。

`AQS`定义了一套多线程访问共享资源的同步模板，解决了实现同步器时涉及的大量细节问题，能够极大地减少实现工作。

**AQS的组成结构：**

三部分组成，`state`同步状态、`Node`组成的`CLH`队列、`ConditionObject`条件变量。

`AbstractQueuedSynchronizer`提供的核心函数：

**状态**

- `getState()`：返回同步状态
- `setState(int newState)`：设置同步状态
- `compareAndSetState(int expect, int update)`：使用`CAS`设置同步状态
- `isHeldExclusively()`：当前线程是否持有资源

**独占资源（不响应线程中断）**

- `tryAcquire(int arg)`：独占式获取资源，子类实现
- `acquire(int arg)`：独占式获取资源模板
- `tryRelease(int arg)`：独占式释放资源，子类实现
- `release(int arg)`：独占式释放资源模板

**共享资源（不响应线程中断）**

- `tryAcquireShared(int arg)`：共享式获取资源，返回值大于等于0则表示获取成功，否则获取失败，子类实现
- `acquireShared(int arg)`：共享形获取资源模板
- `tryReleaseShared(int arg)`：共享式释放资源，子类实现
- `releaseShared(int arg)`：共享式释放资源模板

**同步状态**

在AQS中维护了一个同步状态变量state，对于AQS来说，线程同步的关键是对state的操作，可以说获取、释放资源是否成功都是由state决定的，比如`state>0`代表可获取资源，否则无法获取，所以state的具体语义由实现者去定义。

- ReentrantLock的state用来表示是否有锁资源
- Semaphore的state用来表示可用信号的个数
- CountDownLatch的state用来表示计数器的值

**CLH队列**

`CLH`是`AQS`内部维护的`FIFO`（**先进先出**）双端双向队列（**方便尾部节点插入**），基于链表数据结构，当一个线程竞争资源失败，就会将等待资源的线程封装成一个`Node`节点，通过`CAS`原子操作插入队列尾部，最终不同的`Node`节点连接组成了一个`CLH`队列，所以说`AQS`通过`CLH`队列管理竞争资源的线程。

**Node内部类**

`Node`是`AQS`的内部类，每个等待资源的线程都会封装成`Node`节点组成`CLH`队列、等待队列。

线程获取资源失败，封装成`Node`节点从`CLH`队列尾部入队并阻塞线程，某线程释放资源时会把`CLH`队列首部`Node`节点关联的线程唤醒再次获取资源。

**条件变量**

`Object`的`wait、notify`函数是配合`Synchronized`锁实现线程间同步协作的功能，`AQS`的`ConditionObject`条件变量也提供这样的功能，通过`ConditionObject`的`await`和`signal`两类函数完成。

不同于`Synchronized`锁，一个`AQS`可以对应多个条件变量，而`Synchronized`只有一个。

当某个线程执行了`ConditionObject`的`await`函数，阻塞当前线程，线程会被封装成`Node`节点添加到条件队列的末端，其他线程执行`ConditionObject`的`signal`函数，会将条件队列头部线程节点转移到`CLH`队列参与竞争资源。

**AQS采用了模板方法设计模式，提供了两类模板：**

一类是独占式模板，另一类是共享形模式，对应的模板函数如下：

- 独占式
  - **`acquire`获取资源**
  - **`release`释放资源**
- 共享式
  - **`acquireShared`获取资源**
  - **`releaseShared`释放资源**

**独占式获取资源**

`acquire`是个模板函数，模板流程就是线程获取共享资源，如果获取资源成功，线程直接返回，否则进入`CLH`队列，直到获取资源成功为止，且整个过程忽略中断的影响。

**JDK里利用AQS类的主要步骤：**

1. 在内部写一个 `Sync` 类，该 `Sync` 类继承 `AbstractQueuedSynchronizer`，即 AQS；
2. 在 `Sync` 类里，根据是否是独占，来重写对应的方法。如果是独占，则重写 `tryAcquire` 和 `tryRelease` 等方法；如果是非独占，则重写 `tryAcquireShared` 和 `tryReleaseShared` 等方法；
3. 实现获取/释放的相关方法，并在里面调用 `AQS` 对应的方法，如果是独占则调用 `acquire` 或 `release` 等方法，非独占则调用 `acquireShared` 或 `releaseShared` 或 `acquireSharedInterruptibly` 等