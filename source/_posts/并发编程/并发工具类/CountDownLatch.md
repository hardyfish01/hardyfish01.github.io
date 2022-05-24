---
title: CountDownLatch
categories: 
- 并发编程
- 并发工具类
---

CountDownLatch翻译为计数锁，其最大的作用是通过计数达到等待的功能，主要有两种形式的等待：

1. 让一组线程在全部启动完成之后，再一起执行（先启动的线程需要阻塞等待后启动的线程，直到一组线程全部都启动完成后，再一起执行）；
2. 主线程等待另外一组线程都执行完成之后，再继续执行。

**CountDownLatch 是不能够重用的，比如已经完成了倒数，那可不可以在下一次继续去重新倒数呢？**

这是做不到的，如果你有这个需求的话，可以考虑使用 CyclicBarrier 或者创建一个新的 CountDownLatch 实例。

CountDownLatch 有两个比较重要的 API，分别是 await 和 countDown，管理着线程能否获得锁和锁的释放（也可以称为对 state 的计数增加和减少）

**案例1：**

举个生活中的例子，那就是运动员跑步的场景，比如在比赛跑步时有 5 个运动员参赛，终点有一个裁判员，什么时候比赛结束呢？

那就是当所有人都跑到终点之后，这相当于裁判员等待 5 个运动员都跑到终点，宣布比赛结束。

我们用代码的形式来写出运动员跑步的场景，代码如下：

```java
public class RunDemo1 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(5);
        ExecutorService service = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            final int no = i + 1;
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep((long) (Math.random() * 10000));
                        System.out.println(no + "号运动员完成了比赛。");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        latch.countDown();
                    }
                }
            };
            service.submit(runnable);
        }
        System.out.println("等待5个运动员都跑完.....");
        latch.await();
        System.out.println("所有人都跑完了，比赛结束。");
    }
}
```

**案例2：**

多个线程等待某一个线程的信号，同时开始执行

这和第一个用法有点相反，我们再列举一个实际的场景，比如在运动会上，刚才说的是裁判员等运动员，现在是运动员等裁判员。

在运动员起跑之前都会等待裁判员发号施令，一声令下运动员统一起跑，我们用代码把这件事情描述出来

```java
public class RunDemo2 {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("运动员有5秒的准备时间");
        CountDownLatch countDownLatch = new CountDownLatch(1);
        ExecutorService service = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            final int no = i + 1;
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    System.out.println(no + "号运动员准备完毕，等待裁判员的发令枪");
                    try {
                        countDownLatch.await();
                        System.out.println(no + "号运动员开始跑步了");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            service.submit(runnable);
        }
        Thread.sleep(5000);
        System.out.println("5秒准备时间已过，发令枪响，比赛开始！");
        countDownLatch.countDown();
    }
}
```

**源码解读**

> await方法

await 我们可以叫做等待，也可以叫做加锁，有两种不同入参的方法

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
// 带有超时时间的，最终都会转化成毫秒
public boolean await(long timeout, TimeUnit unit)
    throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

两个方法底层使用的都是 sync，sync 是一个同步器，是 CountDownLatch 的内部类实现的

```
private static final class Sync extends AbstractQueuedSynchronizer {}
```

可以看出来 Sync 继承了 AbstractQueuedSynchronizer，具备了同步器的通用功能。

无参 await 底层使用的是 acquireSharedInterruptibly 方法，有参的使用的是 tryAcquireSharedNanos 方法，这两个方法都是 AQS 的方法，底层实现很相似，主要分成两步：

1. 使用子类的 tryAcquireShared 方法尝试获得锁，如果获取了锁直接返回，获取不到锁走 2；
2. 获取不到锁，用 Node 封装一下当前线程，追加到同步队列的尾部，等待在合适的时机去获得锁。

第二步是 AQS 已经实现了，第一步 tryAcquireShared 方法是交给 Sync 实现的。

```java
// 如果当前同步器的状态是 0 的话，表示可获得锁
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```

获得锁的代码也很简单，直接根据同步器的 state 字段来进行判断，但还是有两点需要注意一下：

1. 获得锁时，state 的值不会发生变化，像 ReentrantLock 在获得锁时，会把 `state + 1`，但 CountDownLatch 不会；
2. CountDownLatch 的 state 并不是 AQS 的默认值 0，而是可以赋值的，是在 CountDownLatch 初始化的时候赋值的

```java
// 初始化,count 代表 state 的初始化值
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    // new Sync 底层代码是 state = count;
    this.sync = new Sync(count);
}
```

这里的初始化的 count 和一般的锁意义不太一样，count 表示我们希望等待的线程数，在两种不同的等待场景中，count 有不同的含义：

1. 让一组线程在全部启动完成之后，再一起执行的等待场景下， count 代表一组线程的个数；
2. 主线程等待另外一组线程都执行完成之后，再继续执行的等待场景下，count 代表一组线程的个数。

所以我们可以把 count 看做我们希望等待的一组线程的个数，可能我们是等待一组线程全部启动完成，可能我们是等待一组线程全部执行完成。

> countDown方法

countDown 中文翻译为倒计时，每调用一次，都会使 state 减一

```java
public void countDown() {
    sync.releaseShared(1);
}
```

releaseShared 是 AQS 定义的方法，方法主要分成两步：

1. 尝试释放锁（tryReleaseShared），锁释放失败直接返回，释放成功走 2；
2. 释放当前节点的后置等待节点。

第二步 AQS 已经实现了，第一步是 Sync 实现的。

```java
// 对 state 进行递减，直到 state 变成 0；
// state 递减为 0 时，返回 true，其余返回 false
protected boolean tryReleaseShared(int releases) {
    // 自旋保证 CAS 一定可以成功
    for (;;) {
        int c = getState();
        // state 已经是 0 了，直接返回 false
        if (c == 0)
            return false;
        // 对 state 进行递减
        int nextc = c-1;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

从源码中可以看到，只有到 count 递减到 0 时，countDown 才会返回 true。