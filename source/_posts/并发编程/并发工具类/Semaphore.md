---
title: Semaphore
categories: 
- 并发编程
- 并发工具类
---

信号量的一个最主要的作用就是，来控制那些需要限制并发访问量的资源。

> 信号量会维护许可证的计数，而线程去访问共享资源前，必须先拿到许可证。
>
> 线程可以从信号量中去获取一个许可证，一旦线程获取之后，信号量持有的许可证就转移过去了，信号量手中剩余的许可证要减一。
>
> 线程也可以释放一个许可证，如果线程释放了许可证，这个许可证相当于被归还给信号量了，信号量中的许可证的可用数量加一。
>
> 当信号量拥有的许可证数量减到 0 时，如果下个线程还想要获得许可证，那么这个线程就必须等待，直到之前得到许可证的线程释放，它才能获取。

**使用流程**

主要分为以下几步：

- 首先初始化一个信号量，并且传入许可证的数量：`public Semaphore(int permits, boolean fair)`，传入两个参数，第一个参数是许可证的数量，另一个参数是是否公平。
- 如果第二个参数传入 true，则代表它是公平的策略，会把之前已经等待的线程放入到队列中，而当有新的许可证到来时，它会把这个许可证按照顺序发放给之前正在等待的线程；
- 如果这个构造函数第二个参数传入 false，则代表非公平策略，也就有可能插队，就是说后进行请求的线程有可能先得到许可证。
- 在调用慢服务之前，让线程来调用 acquire 方法或者 acquireUninterruptibly方法，这两个方法的作用是要获取许可证，这同时意味着只有这个方法能顺利执行下去的话，它才能进一步访问这个代码后面的调用慢服务的方法。
- 如果此时信号量已经没有剩余的许可证了，那么线程就会等在 acquire 方法的这一行代码中，所以它也不会进一步执行下面调用慢服务的方法。
- 在任务执行完毕之后，调用 release() 来释放许可证，比如说我们在执行完慢服务这行代码之后，再去执行 release() 方法，这样一来，许可证就会还给我们的信号量了。

**示例代码**

```java
public class SemaphoreDemo {
    static Semaphore semaphore = new Semaphore(3);
    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(50);
        for (int i = 0; i < 1000; i++) {
            service.submit(new Task());
        }
        service.shutdown();
    }
    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                semaphore.acquire();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "拿到了许可证，花费2秒执行慢服务");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("慢服务执行完毕，" + Thread.currentThread().getName() + "释放了许可证");
            semaphore.release();
        }
    }
}
```

在这段代码中我们新建了一个数量为 3 的信号量，然后又有一个和之前一样的固定 50 线程的线程池，并且往里面放入 1000 个任务。

每个任务在执行模拟慢服务之前，会先用信号量的 acquire 方法获取到信号量，然后再去执行这 2 秒钟的慢服务，最后利用 release() 方法来释放许可证。

**特殊用法：一次性获取或释放多个许可证**

比如 `semaphore.acquire(2)`，里面传入参数 2，这就叫一次性获取两个许可证。

同时释放也是一样的，`semaphore.release(3)` 相当于一次性释放三个许可证。

**信号量能被 FixedThreadPool 替代吗？**

信号量具有**跨线程、跨线程池**的特性，所以即便这些请求来自于不同的线程池，我们也可以限制它们的访问。

如果用 FixedThreadPool 去限制，那就做不到跨线程池限制了，这样的话会让功能大大削弱。