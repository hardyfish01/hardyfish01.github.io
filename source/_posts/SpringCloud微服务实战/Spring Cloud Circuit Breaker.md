---
title: Spring Cloud Circuit Breaker
categories: 
- SpringCloud微服务实战
---

Spring Cloud 中专门用于提供服务容错功能的 Spring Cloud Circuit Breaker 框架。

从命名上看，Spring Cloud Circuit Breaker 是对熔断器的一种抽象，支持不同的熔断器实现方案。

**服务熔断**

<img src="https://img-blog.csdnimg.cn/bb0d9c139e014052bf869fd165453d47.png" style="zoom:25%;" />

* Closed： 对于熔断器而言，Closed 状态代表熔断器不进行任何的熔断处理。尽管这个时候人们感觉不到熔断器的存在，但它在背后会对调用失败次数进行积累，到达一定阈值或比例时则自动启动熔断机制。

* Open： 一旦对服务的调用失败次数达到一定阈值时，熔断器就会打开，这时候对服务的调用将直接返回一个预定的错误，而不执行真正的网络调用。同时，熔断器内置了一个时间间隔，当处理请求达到这个时间间隔时会进入半熔断状态。

* Half-Open： 在半开状态下，熔断器会对通过它的部分请求进行处理，如果对这些请求的成功处理数量达到一定比例则认为服务已恢复正常，就会关闭熔断器，反之就会打开熔断器。