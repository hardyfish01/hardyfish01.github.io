---
title: Timer
categories: 
- Java基础
---

**Timer和TimerTask**

Timer 是 JDK 自带的执行定时任务的工具类，用户能够指定延迟、任务执行的时间间隔。

当 new 一个 Timer 时，会自动启动一个 TimerTask 线程，这个线程无限循环一个小顶堆的任务队列，每次取出最近需要执行的任务，如果符合条件则对该任务（小顶堆）做相应处理。

**Timer的使用**

```java
public static void main(String[] args) {
    Timer timer = new Timer();
    timer.scheduleAtFixedRate(new TimerTask() {
        
        @Override
        public void run() {
            System.out.println(new Date(this.scheduledExecutionTime()));
        }
        
    }, 500L, 1000L);
}
```

**缺陷**

任务是串行的，如果前一个任务耗时较久，就会影响后面任务的执行。

如果在定时任务(TimerTask)抛出了异常会终止定时器的执行。那么当第一个任务抛出了异常，第二个任务也不会被调度执行。

