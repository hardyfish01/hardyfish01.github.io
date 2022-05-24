---
title: Eureka
categories: 
- 常用框架
- Spring
- SpringCloud
---

Netflix Eureka 是一款由 Netflix 开源的基于 REST 服务的注册中心，用于提供服务发现功能。

Spring Cloud Eureka 是 Spring Cloud Netflix 微服务套件的一部分，基于 Netflix Eureka 进行了二次封装，主要负责完成微服务架构中的服务治理功能。

**架构剖析**

Eureka 的架构主要分为 Eureka Server 和 Eureka Client 两部分，Eureka Client 又分为 Applicaton Service 和 Application Client，Applicaton Service 就是服务提供者，Application Client 就是服务消费者。

- 我们首先会在应用程序中依赖 Eureka Client，项目启动后 Eureka Client 会向 Eureka Server 发送请求，进行注册，并将自己的一些信息发送给 Eureka Server。
- 注册成功后，每隔一定的时间，Eureka Client 会向 Eureka Server 发送心跳来续约服务，也就是汇报健康状态。
- 如果客户端长时间没有续约，那么 Eureka Server 大约将在 90 秒内从服务器注册表中删除客户端的信息。
- Eureka Client 还会定期从 Eureka Server 拉取注册表信息，然后根据负载均衡算法得到一个目标，并发起远程调用。
- 应用停止时也会通知 Eureka Server 移除相关信息，信息成功移除后，对应的客户端会更新服务的信息，这样就不会调用已经下线的服务了，当然这个会有延迟，有可能会调用到已经失效的服务，所以在客户端会开启失败重试功能来避免这个问题。
- Eureka Server 会有多个节点组成一个集群，保证高可用。
- Eureka Server 没有集成其他第三方存储，而是存储在内存中。

所以 Eureka Server 之间会将注册信息复制到集群中的 Eureka Server 的所有节点。

这样数据才是共享状态，任何的 Eureka Client 都可以在任何一个 Eureka Server 节点查找注册表信息。

![img](https://img-blog.csdnimg.cn/b2d2e0f441684ed9b7256ee46cf78dbc.png)

**自我保护机制**

默认情况下，如果 Eureka Server 在一定的 90s 内没有接收到某个微服务实例的心跳，会注销该实例。

但是在微服务架构下服务之间通常都是跨进程调用，网络通信往往会面临着各种问题，比如微服务状态正常，网络分区故障，导致此实例被注销。

固定时间内大量实例被注销，可能会严重威胁整个微服务架构的可用性。

为了解决这个问题，Eureka 开发了自我保护机制，那么什么是自我保护机制呢？

Eureka Server 在运行期间会去统计心跳失败比例在 15 分钟之内是否低于 85%，如果低于 85%，Eureka Server 即会进入自我保护机制。

**Eureka Server 触发自我保护机制后，页面会出现提示：**

![img](https://img-blog.csdnimg.cn/2cf55a55e55c44649dfd609ab758c2c1.png)

**Eureka Server 进入自我保护机制，会出现以下几种情况：**

- Eureka 不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
- Eureka 仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

> Eureka 自我保护机制是为了防止误杀服务而提供的一个机制。

当个别客户端出现心跳失联时，则认为是客户端的问题，剔除掉客户端；当 Eureka 捕获到大量的心跳失败时，则认为可能是网络问题，进入自我保护机制；当客户端心跳恢复时，Eureka 会自动退出自我保护机制。

如果在保护期内刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，即会调用失败。

对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。

**通过在 Eureka Server 配置如下参数，开启或者关闭保护机制，生产环境建议打开：**

```
eureka.server.enable-self-preservation=true
```