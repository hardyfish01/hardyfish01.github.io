---
title: Future
categories: 
- 公众号文章
---

文章首发在公众号（月伴飞鱼），之后同步到掘金和个人网站：http://xiaoflyfish.cn/

**觉得有收获，希望帮忙点赞，转发下哈，谢谢，谢谢**

# 前言

Java 5在concurrency包中引入了`java.util.concurrent.Callable` 接口，它和Runnable接口很相似，但它可以返回一个对象或者抛出一个异常。

Callable接口使用泛型去定义它的返回类型。Executors类提供了一些有用的方法在线程池中执行Callable内的任务。由于Callable任务是并行的，我们必须等待它返回的结果。而线程是属于异步计算模型，所以不可能直接从别的线程中得到函数返回值。

`java.util.concurrent.Future`对象为我们解决了这个问题。在线程池提交Callable任务后返回了一个Future对象，使用它可以知道Callable任务的状态和得到Callable返回的执行结果。Future提供了get()方法让我们可以等待Callable结束并获取它的执行结果。

# Future的作用

当做一定运算的时候，运算过程可能比较耗时，有时会去查数据库，或是繁重的计算，比如压缩、加密等，在这种情况下，如果我们一直在原地等待方法返回，显然是不明智的，整体程序的运行效率会大大降低。

我们可以把运算的过程放到子线程去执行，再通过 Future 去控制子线程执行的计算过程，最后获取到计算结果。

这样一来就可以把整个程序的运行效率提高，是一种**异步**的思想。

同时在JDK 1.8的doc中，对Future的描述如下：

> A Future represents the result of an asynchronous computation. Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation.

大概意思就是Future是一个用于**异步计算**的接口。

**举个例子：**

比如去吃早点时，点了包子和凉菜，包子需要等3分钟，凉菜只需1分钟，如果是串行的一个执行，在吃上早点的时候需要等待4分钟，但是如果你在准备包子的时候，可以同时准备凉菜，这样只需要等待3分钟。

Future就是后面这种执行模式。

**之前我写的一篇文章：**[实现异步编程，这个工具类你得掌握！](https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247485100&idx=1&sn=3c00ec90b2accb812cf5de5e2eeabd96&scene=21#wechat_redirect)，详细的写了关于Future这个的应用

# 创建Future

## 线程池

```java
class Task implements Callable<String> {
  public String call() throws Exception {
    return longTimeCalculation(); 
  } 
}
ExecutorService executor = Executors.newFixedThreadPool(4); 
// 定义任务:
Callable<String> task = new Task(); 
// 提交任务并获得Future: 
Future<String> future = executor.submit(task); 
// 从Future获取异步执行返回的结果: 
String result = future.get(); // 可能阻塞
```

当我们提交一个Callable任务后，我们会同时获得一个Future对象，然后，我们在主线程某个时刻调用Future对象的get()方法，就可以获得异步执行的结果。

在调用get()时，如果异步任务已经完成，我们就直接获得结果。如果异步任务还没有完成，那么get()会阻塞，直到任务完成后才返回结果

## FutureTask

除了用线程池的 submit 方法会返回一个 future 对象之外，同样还可以用 FutureTask 来获取 Future 类和任务的结果。

**我们来看一下 FutureTask 的代码实现：**

```java
public class FutureTask<V> implements RunnableFuture<V>{
 ...
}
```

可以看到，它实现了一个接口，这个接口叫作 RunnableFuture。

我们再来看一下 RunnableFuture 接口的代码实现：

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

既然 RunnableFuture 继承了 Runnable 接口和 Future 接口，而 FutureTask 又实现了 RunnableFuture 接口，所以 FutureTask 既可以作为 Runnable 被线程执行，又可以作为 Future 得到 Callable 的返回值。

典型用法是，把 Callable 实例当作 FutureTask 构造函数的参数，生成 FutureTask 的对象，然后把这个对象当作一个 Runnable 对象，放到线程池中或另起线程去执行，最后还可以通过 FutureTask 获取任务执行的结果。

**下面我们就用代码来演示一下：**

```java
public class FutureTaskDemo {

    public static void main(String[] args) {
        Task task = new Task();
        FutureTask<Integer> integerFutureTask = new FutureTask<>(task);
        new Thread(integerFutureTask).start();

        try {
            System.out.println("task运行结果："+integerFutureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class Task implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        System.out.println("子线程正在计算");
        int sum = 0;
        for (int i = 0; i < 100; i++) {
            sum += i;
        }
        return sum;
    }
}
```

在这段代码中可以看出，首先创建了一个实现了 Callable 接口的 Task，然后把这个 Task 实例传入到 FutureTask 的构造函数中去，创建了一个 FutureTask 实例，并且把这个实例当作一个 Runnable 放到 new Thread() 中去执行，最后再用 FutureTask 的 get 得到结果，并打印出来。

# Future常用方法

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43d661647583424c8f2b9ca1b37a295c~tplv-k3u1fbpfcp-zoom-1.image)

| 方法名      | 返回值  | 入参                              | 备注                                                         | 总结                                                         |
| ----------- | ------- | --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| cancel      | boolean | （boolean mayInterruptIfRunning） | 用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。 | 也就是说Future提供了三种功能：判断任务是否完成，能够中断任务，能够获取任务执行结果 |
| isCancelled | boolean | 无                                | 方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。 |                                                              |
| isDone      | boolean | 无                                | 方法表示任务是否已经完成，若任务完成，则返回true；           |                                                              |
| get         | V       | 无                                | 方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回 |                                                              |
| get         | V       | （long timeout, TimeUnit unit）   | 用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null |                                                              |

## get()方法

get方法最主要的作用就是获取任务执行的结果

**我们来看一个代码示例：**

```java
public class FutureTest {

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(10);
        Future<Integer> future = service.submit(new CallableTask());
        try {
            System.out.println(future.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        service.shutdown();
    }

    static class CallableTask implements Callable<Integer> {

        @Override
        public Integer call() throws Exception {
            Thread.sleep(3000);
            return new Random().nextInt();
        }
    }
}
```

在这段代码中，main 方法新建了一个 10 个线程的线程池，并且用 submit 方法把一个任务提交进去。

这个任务它所做的内容就是先休眠三秒钟，然后返回一个随机数。

接下来我们就直接把`future.get`结果打印出来，其结果是正常打印出一个随机数，比如 9527 等。

## isDone()方法

该方法是用来判断当前这个任务是否执行完毕了。

需要注意的是，这个方法如果返回 true 则代表执行完成了；如果返回 false 则代表还没完成。

但这里如果返回 true，并不代表这个任务是成功执行的，比如说任务执行到一半抛出了异常。那么在这种情况下，对于这个 isDone 方法而言，它其实也是会返回 true 的，因为对它来说，虽然有异常发生了，但是这个任务在未来也不会再被执行，它确实已经执行完毕了。

所以 isDone 方法在返回 true 的时候，不代表这个任务是成功执行的，只代表它执行完毕了。

**我们用一个代码示例来看一看，代码如下所示：**

```java
public class GetException {

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(20);
        Future<Integer> future = service.submit(new CallableTask());

        try {
            for (int i = 0; i < 5; i++) {
                System.out.println(i);
                Thread.sleep(500);
            }
            System.out.println(future.isDone());
            future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    static class CallableTask implements Callable<Integer> {

        @Override
        public Integer call() throws Exception {
            throw new IllegalArgumentException("Callable抛出异常");
        }
    }
}
```

在这段代码中，可以看到有一个线程池，并且往线程池中去提交任务，这个任务会直接抛出一个异常。

那么接下来我们就用一个 for 循环去休眠，同时让它慢慢打印出 0 ~ 4 这 5 个数字，这样做的目的是起到了一定的延迟作用。

在这个执行完毕之后，再去调用 isDone() 方法，并且把这个结果打印出来，然后再去调用 `future.get()`

## cancel方法

如果不想执行某个任务了，则可以使用 cancel 方法，会有以下三种情况：

- 第一种情况最简单，那就是当任务还没有开始执行时，一旦调用 cancel，这个任务就会被正常取消，未来也不会被执行，那么 cancel 方法返回 true。
- 第二种情况也比较简单。如果任务已经完成，或者之前已经被取消过了，那么执行 cancel 方法则代表取消失败，返回 false。因为任务无论是已完成还是已经被取消过了，都不能再被取消了。
- 第三种情况就是这个任务正在执行，这个时候会根据我们传入的参数mayInterruptIfRunning做判断，如果传入的参数是 true，执行任务的线程就会收到一个中断的信号，正在执行的任务可能会有一些处理中断的逻辑，进而停止，如果传入的是 false 则就代表不中断正在运行的任务

## isCancelled()方法

判断是否被取消，它和 cancel 方法配合使用，比较简单。

# 应用场景

目前对于Future方式，我们经常使用的有这么几类：

**Guava**

ListenableFutrue，通过增加监听器的方式，计算完成时立即得到结果，而无需一直循环查询

**CompletableFuture**

Java8的CompletableFuture，使用thenApply，thenApplyAsync可以达到和Guava类似的链式调用效果。

不同的是，对于Java8，如果thenApplyAsync不传入线程池，则会使用ForkJoinPools线程池来执行对应的方法，如此可以避免对其他线程产生影响。

**之前我写的一篇文章：**[实现异步编程，这个工具类你得掌握！](https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247485100&idx=1&sn=3c00ec90b2accb812cf5de5e2eeabd96&scene=21#wechat_redirect)，详细的写了关于Future这个的应用

**Netty**

Netty解决的问题：

- 原生Future的isDone()方法判断一个异步操作是否完成，但是定义比较模糊：正常终止、抛出异常、用户取消都会使isDone方法返回true。
- 对于一个异步操作，我们有些时候更关注的是这个异步操作触发或者结束后能否再执行一系列的动作。

与JDK相比，增加了完成状态的细分，增加了监听者，异步线程结束之后能够触发一系列的动作。

# 注意事项

## 添加超时机制

假设一共有四个任务需要执行，我们都把它放到线程池中，然后它获取的时候是按照从 1 到 4 的顺序，也就是执行 get() 方法来获取的

代码如下所示：

```java
public class FutureDemo {


    public static void main(String[] args) {
        //创建线程池
        ExecutorService service = Executors.newFixedThreadPool(10);
        //提交任务，并用 Future 接收返回结果
        ArrayList<Future> allFutures = new ArrayList<>();
        for (int i = 0; i < 4; i++) {
            Future<String> future;
            if (i == 0 || i == 1) {
                future = service.submit(new SlowTask());
            } else {
                future = service.submit(new FastTask());
            }
            allFutures.add(future);
        }

        for (int i = 0; i < 4; i++) {
            Future<String> future = allFutures.get(i);
            try {
                String result = future.get();
                System.out.println(result);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
        service.shutdown();
    }

    static class SlowTask implements Callable<String> {

        @Override
        public String call() throws Exception {
            Thread.sleep(5000);
            return "速度慢的任务";
        }
    }

    static class FastTask implements Callable<String> {

        @Override
        public String call() throws Exception {
            return "速度快的任务";
        }
    }
}
```

可以看出，在代码中我们新建了线程池，并且用一个 list 来保存 4 个 Future。

其中，前两个 Future 所对应的任务是慢任务，也就是代码下方的 SlowTask，而后两个 Future 对应的任务是快任务。

慢任务在执行的时候需要 5 秒钟的时间才能执行完毕，而快任务很快就可以执行完毕，几乎不花费时间。

在提交完这 4 个任务之后，我们用 for 循环对它们依次执行 get 方法，来获取它们的执行结果，然后再把这个结果打印出来。

实际上在执行的时候会先等待 5 秒，然后再很快打印出这 4 行语句。

**所以问题是：**

第三个的任务量是比较小的，它可以很快返回结果，紧接着第四个任务也会返回结果。

但是由于前两个任务速度很慢，所以我们在利用 get 方法执行时，会卡在第一个任务上。也就是说，虽然此时第三个和第四个任务很早就得到结果了，但我们在此时使用这种 for 循环的方式去获取结果，依然无法及时获取到第三个和第四个任务的结果。直到 5 秒后，第一个任务出结果了，我们才能获取到，紧接着也可以获取到第二个任务的结果，然后才轮到第三、第四个任务。

假设由于网络原因，第一个任务可能长达 1 分钟都没办法返回结果，那么这个时候，我们的主线程会一直卡着，影响了程序的运行效率。

此时我们就可以用 Future 的带超时参数的`get(long timeout, TimeUnit unit)`方法来解决这个问题。

这个方法的作用是，如果在限定的时间内没能返回结果的话，那么便会抛出一个 TimeoutException 异常，随后就可以把这个异常捕获住，或者是再往上抛出去，这样就不会一直卡着了。

# 源码分析

## 超时实现原理

具体实现类：FutureTask

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/644a12e1f7d747dfb6bbf69fbda9a2e9~tplv-k3u1fbpfcp-zoom-1.image)

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43db7024a9424ecab4c4a19d35200c7b~tplv-k3u1fbpfcp-zoom-1.image)

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed45c4fe5f414c40911b5c2d8443da2d~tplv-k3u1fbpfcp-zoom-1.image)

**get()方法可以分为两步：**

- 判断当前任务的执行状态，如果不是COMPLETING，就调用awaitDone()方法开始进行死循环轮旋，如果任务还没有执行完成会使用`nanos = deadline - System.nanoTime()`检查是否超时，如果方法已经超时，则会返回，在返回后如果任务的状态仍然`<=COMPLETING`，就会抛出TimeoutException()。
- 如果调用时任务没有执行完成，会调用parkNanos()，调用线程会阻塞在这里。

**接下来分两种情况：**

1. 在阻塞时间完以后任务的执行状态仍然没有改变为完成，进入下一次循环，直接返回。
2. 如果在轮询中状态已经改变，任务完成，则会中断死循环，返回任务执行的返回值。

# 最后

**觉得有收获，希望帮忙点赞，转发下哈，谢谢，谢谢**

微信搜索：月伴飞鱼，交个朋友，进面试交流群

公众号后台回复666，可以获得免费电子书籍