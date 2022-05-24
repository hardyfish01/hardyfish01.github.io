---
title: Ribbon
categories: 
- 常用框架
- Spring
- SpringCloud
---

Ribbon 就是负载均衡的组件，所有的请求都是通过 Ribbon 来选取对应的服务信息的。

**目前主流的负载方案分为两种：**

- 集中式负载均衡，在消费者和服务提供方中间使用独立的代理方式进行负载，有硬件的负载均衡器，比如 F5，也有软件，比如 Nginx。
- 客户端负载均衡，客户端根据自己的请求情况做负载，Ribbon 就属于客户端自己做负载的框架。

Ribbon 是由 Netflix 发布的负载均衡器，它有助于控制 HTTP 和 TCP 的客户端的行为。

Ribbon 属于客户端负载均衡。

- 为 Ribbon 配置服务提供者地址后，Ribbon 就可基于某种负载均衡算法，自动的帮助服务消费者进行请求。
- 同时 Ribbon 默认为我们提供了很多负载均衡算法，例如：轮询、随机算法等。

因为 Netflix Ribbon 本质上只是一个工具，而不是一套完整的解决方案，所以 Spring Cloud Netflix Ribbon 对 Netflix Ribbon 做了封装和集成，使其可以融入以 Spring Boot 为构建基础的技术体系中。

**主要组件**

<img src="https://img-blog.csdnimg.cn/d2ecf9e82137430fa123900844fb0ed5.png" alt="img" style="zoom:25%;" />

需要一个服务实例的存储组件来支持，ServerList 就是这个组件。

- 存储分为静态和动态两种方式。静态存储需要事先配置好固定的服务实例信息，动态存储需要从注册中心获取对应的服务实例信息。
- 有了服务信息后，在某些场景下我们可能需要过滤一部分信息，这个时候可以用 ServerListFilter 组件来实现过滤操作。

Ribbon 会将服务实例在本地内存中存储一份，这样就不需要每次都去注册中心获取信息，这种场景的问题在于当服务实例增加或者减少后，本地怎么更新呢？

这个时候就需要用到ServerListUpdater 组件，ServerListUpdater 组件就是用于服务实例更新操作。

还有个问题就是，缓存到本地的服务实例信息有可能已经无法提供服务了，这个时候就需要有一个检测的组件，来检测服务实例信息是否可用，这个组件就是 IPing。

Ribbon 会根据指定的算法来选择一个可用的实例信息，IRule 组件提供了很多种算法策略来选择实例信息。

最后就是我们使用的入口了，我们要选择一个可用的服务，怎么选择？问谁要这个服务？

这时ILoadBalancer 就上场了，ILoadBalancer 中定义了软件负载均衡操作的接口，比如动态更新一组服务列表，根据指定算法从现有服务器列表中选择一个可用的服务等操作。

<img src="https://img-blog.csdnimg.cn/a8b528619cea4d7998989b1615b6d7e0.png" alt="img" style="zoom:33%;" />

**内置负载均衡策略**

IRule 是算法的接口。AbstractLoadBalancerRule 是实现了 IRule 接口的抽象类，所有内置的算法都是继承 AbstractLoadBalancerRule 来实现的。

- RoundRobinRule 是轮询的算法，如果有 A、B 两个实例，那么该算法的逻辑是选择 A，再选择B，再选择A，轮询下去。
- RandomRule 是随机算法，这个就比较简单了，在服务列表中随机选取。
- BestAvailableRule 选择一个最小的并发请求 server，如果有 A、B 两个实例，当 A 有 4 个请求正在处理中，B 有 2 个请求正在处理中，下次选择的时候会选择 B，因为 B 处理的数量是最少的，认为它压力最小，这种场景适合于服务所在机器的配置都相同的情况下，否则不太适用。
- 在某些场景下，服务器的性能不一致，如果采用轮询算法或随机算法的话，无法充分的利用服务器的资源，某些服务器处理的快，应该多分配一些请求，某些服务器处理的慢，就可以少分配一些请求，WeightedResponseTimeRule 就是做这件事情的，原理是根据请求的响应时间计算权重，如果响应时间越长，那么对应的权重越低，权重越低的服务器，被选择的可能性就越低。

**负载均衡设置**

以 Nacos 中的 Ribbon 负载均衡设置为例，在配置文件 `application.yml` 中设置如下配置即可：

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #设置负载均衡策略
```