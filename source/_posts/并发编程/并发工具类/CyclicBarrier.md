---
title: CyclicBarrier
categories: 
- 并发编程
- 并发工具类
---

CyclicBarrier 可以构造出一个集结点，当某一个线程执行 `await()` 的时候，它就会到这个集结点开始等待，等待这个栅栏被撤销。

直到预定数量的线程都到了这个集结点之后，这个栅栏就会被撤销，之前等待的线程就在此刻统一出发，继续去执行剩下的任务。

**举一个生活中的例子**

假设我们班级春游去公园里玩，并且会租借三人自行车，每个人都可以骑，但由于这辆自行车是三人的，所以要凑齐三个人才能骑一辆。

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3);
        for (int i = 0; i < 6; i++) {
            new Thread(new Task(i + 1, cyclicBarrier)).start();
        }
    }
    static class Task implements Runnable {
        private int id;
        private CyclicBarrier cyclicBarrier;
        public Task(int id, CyclicBarrier cyclicBarrier) {
            this.id = id;
            this.cyclicBarrier = cyclicBarrier;
        }
        @Override
        public void run() {
            System.out.println("同学" + id + "现在从大门出发，前往自行车驿站");
            try {
                Thread.sleep((long) (Math.random() * 10000));
                System.out.println("同学" + id + "到了自行车驿站，开始等待其他人到来");
                cyclicBarrier.await();
                System.out.println("同学" + id + "开始骑车");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

首先建了一个参数为 3 的 CyclicBarrier，参数为 3 的意思是需要等待 3 个线程到达这个集结点才统一放行。

然后我们又在 for 循环中去开启了 6 个线程，每个线程中执行的 Runnable 对象就在下方的 Task 类中。

当同学们都到了驿站之后，比如某一个同学到了驿站，首先会打印出同学某某到了自行车驿站，开始等待其他人到达的消息，然后去调用 CyclicBarrier 的 await() 方法。

一旦它调用了这个方法，它就会陷入等待，直到三个人凑齐，才会继续往下执行，一旦开始继续往下执行，就意味着 3 个同学开始一起骑车了。

**执行结果如下：**

```
同学 1 现在从大门出发，前往自行车驿站
同学 5 现在从大门出发，前往自行车驿站
同学 6 现在从大门出发，前往自行车驿站
同学 4 现在从大门出发，前往自行车驿站
同学 3 现在从大门出发，前往自行车驿站
同学 2 现在从大门出发，前往自行车驿站
同学 3到了自行车驿站，开始等待其他人到来
同学 2到了自行车驿站，开始等待其他人到来
同学 6到了自行车驿站，开始等待其他人到来
同学 6开始骑车
同学 2开始骑车
同学 3开始骑车
同学 1到了自行车驿站，开始等待其他人到来
同学 5到了自行车驿站，开始等待其他人到来
同学 4到了自行车驿站，开始等待其他人到来
同学 4开始骑车
同学 1开始骑车
同学 5开始骑车
```

**执行动作barrierAction**

```
public CyclicBarrier(int parties, Runnable barrierAction)：
```

第一个参数依然是 parties，代表需要几个线程到齐；第二个参数是一个 Runnable 对象，它就是barrierAction。

当预设数量的线程到达了集结点之后，在出发的时候，便会执行这里所传入的 Runnable 对象，比如：

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(3, new Runnable() {
    @Override
    public void run() {
        System.out.println("凑齐3人了，出发！");
    }
});
```

这个语句每个周期只打印一次，不是说你有几个线程在等待就打印几次，而是说这个任务只在开闸的时候执行一次。

**CyclicBarrier 和 CountDownLatch 的异同**

相同点：都能阻塞一个或一组线程，直到某个预设的条件达成发生，再统一出发。

不同点，具体如下：

- 作用对象不同： CyclicBarrier 要等固定数量的线程都到达了栅栏位置才能继续执行，而 CountDownLatch 只需等待数字倒数到 0，也就是说 CountDownLatch 作用于事件。
- 可重用性不同： CountDownLatch 在倒数到 0 并且触发门闩打开后，就不能再次使用了，除非新建一个新的实例；而 CyclicBarrier 可以重复使用，CyclicBarrier 还可以随时调用 reset 方法进行重置，如果重置时有线程已经调用了 await 方法并开始等待，那么这些线程则会抛出 BrokenBarrierException 异常。
- 执行动作不同： CyclicBarrier 有执行动作 barrierAction，而 CountDownLatch 没这个功能。