---
title: Spring Cloud Gateway
categories: 
- SpringCloud微服务实战
---

Spring Cloud Gateway 是 Spring 官方自己开发的一款 API 网关。

* Zuul 的实现原理是对 Servlet 的一层封装，通信模式上采用的是阻塞式 I/O。

而在技术体系上，Spring Cloud Gateway 基于最新的 Spring 5 和 Spring Boot 2，以及用于响应式编程的 Project Reactor 框架，提供的是响应式、非阻塞式 I/O 模型。

* 所以较之 Netflix Zuul，性能上Spring Cloud Gateway 显然要更胜一筹。

另一方面，从功能上，Spring Cloud Gateway 也比 Zuul 更为丰富。除了通用的服务路由机制之外，Spring Cloud Gateway 还支持请求限流等面向服务容错方面的功能，同样也能与 Hystrix 等框架进行良好的集成。

**基本架构**

Spring Cloud Gateway 中的核心概念有两个，一个是过滤器（Filter），一个是谓词（Predicate）。

Spring Cloud Gateway 的整体架构图如下图所示：

<img src="https://img-blog.csdnimg.cn/8de47fd1f44a4e068b930702dfc47beb.png" style="zoom:25%;" />

Spring Cloud Gateway 中的过滤器和 Zuul 中的过滤器是同一个概念。

* 它们都可以用于在处理 HTTP 请求之前或之后修改请求本身，及对应响应结果。

* 区别在于两者的类型和实现方式不同。

而所谓谓词，本质上是一种判断条件，用于将 HTTP 请求与路由进行匹配。

Spring Cloud Gateway 内置了大量的谓词组件，可以分别对 HTTP 请求的消息头、请求路径等常见的路由媒介进行自动匹配以便决定路由结果。
