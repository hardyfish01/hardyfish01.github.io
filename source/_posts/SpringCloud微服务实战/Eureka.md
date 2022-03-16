---
title: Eureka
categories: 
- SpringCloud微服务实战
---

Netflix Eureka 是一款由 Netflix 开源的基于 REST 服务的注册中心，用于提供服务发现功能。

Spring Cloud Eureka 是 Spring Cloud Netflix 微服务套件的一部分，基于 Netflix Eureka 进行了二次封装，主要负责完成微服务架构中的服务治理功能。

**架构剖析**

Eureka 的架构主要分为 Eureka Server 和 Eureka Client 两部分，Eureka Client 又分为 Applicaton Service 和 Application Client，Applicaton Service 就是服务提供者，Application Client 就是服务消费者。

* 我们首先会在应用程序中依赖 Eureka Client，项目启动后 Eureka Client 会向 Eureka Server 发送请求，进行注册，并将自己的一些信息发送给 Eureka Server。

* 注册成功后，每隔一定的时间，Eureka Client 会向 Eureka Server 发送心跳来续约服务，也就是汇报健康状态。 

* 如果客户端长时间没有续约，那么 Eureka Server 大约将在 90 秒内从服务器注册表中删除客户端的信息。 

* Eureka Client 还会定期从 Eureka Server 拉取注册表信息，然后根据负载均衡算法得到一个目标，并发起远程调用。

* 应用停止时也会通知 Eureka Server 移除相关信息，信息成功移除后，对应的客户端会更新服务的信息，这样就不会调用已经下线的服务了，当然这个会有延迟，有可能会调用到已经失效的服务，所以在客户端会开启失败重试功能来避免这个问题。

* Eureka Server 会有多个节点组成一个集群，保证高可用。

* Eureka Server 没有集成其他第三方存储，而是存储在内存中。

所以 Eureka Server 之间会将注册信息复制到集群中的 Eureka Server 的所有节点。 

这样数据才是共享状态，任何的 Eureka Client 都可以在任何一个 Eureka Server 节点查找注册表信息。

<img src="https://img-blog.csdnimg.cn/042a0285aaea4ccabd0a860878dccc2d.png" style="zoom:25%;" />

**Eureka Server：注册中心服务端**

注册中心服务端主要对外提供了三个功能：

**服务注册**

服务提供者启动时，会通过 Eureka Client 向 Eureka Server 注册信息，Eureka Server 会存储该服务的信息，Eureka Server 内部有缓存机制来维护整个注册表

**提供注册表**

服务消费者在调用服务时，如果 Eureka Client 没有缓存注册表的话，会从 Eureka Server 获取最新的注册表

**同步状态**

Eureka Client 通过注册、心跳机制和 Eureka Server 同步当前客户端的状态。

**Eureka Client：注册中心客户端**

Eureka Client 是一个 Java 客户端，用于简化与 Eureka Server 的交互。

Eureka Client 会拉取、更新和缓存 Eureka Server 中的信息。因此当所有的 Eureka Server 节点都宕掉，服务消费者依然可以使用缓存中的信息找到服务提供者，但是当服务有更改的时候会出现信息不一致。

**Register: 服务注册**

服务的提供者，将自身注册到注册中心，服务提供者也是一个 Eureka Client。

当 Eureka Client 向 Eureka Server 注册时，它提供自身的元数据，比如 IP 地址、端口，运行状况指示符 URL，主页等。

**Renew: 服务续约**

Eureka Client 会每隔 30 秒发送一次心跳来续约。 通过续约来告知 Eureka Server 该 Eureka Client 运行正常，没有出现问题。 

默认情况下，如果 Eureka Server 在 90 秒内没有收到 Eureka Client 的续约，Server 端会将实例从其注册表中删除，此时间可配置，一般情况不建议更改。

```properties
#服务续约任务的调用间隔时间，默认为30秒
eureka.instance.lease-renewal-interval-in-seconds=30

#服务失效的时间，默认为90秒。
eureka.instance.lease-expiration-duration-in-seconds=90
```

**GetRegisty: 获取注册列表信息**

* Eureka Client 从服务器获取注册表信息，并将其缓存在本地。

* 客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。

* 每次返回注册列表信息可能与 Eureka Client 的缓存信息不同，Eureka Client 自动处理。

* 如果由于某种原因导致注册列表信息不能及时匹配，Eureka Client 则会重新获取整个注册表信息。

* Eureka Server 缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩。

* Eureka Client 和 Eureka Server 可以使用 JSON/XML 格式进行通讯。

在默认情况下 Eureka Client 使用压缩 JSON 格式来获取注册列表的信息。

```properties
# 启用服务消费者从注册中心拉取服务列表的功能
eureka.client.fetch-registry=true

# 设置服务消费者从注册中心拉取服务列表的间隔
eureka.client.registry-fetch-interval-seconds=30
```

**Remote Call: 远程调用**

当 Eureka Client 从注册中心获取到服务提供者信息后，就可以通过 Http 请求调用对应的服务；

服务提供者有多个时，Eureka Client 客户端会通过 Ribbon 自动进行负载均衡。

**自我保护机制**

默认情况下，如果 Eureka Server 在一定的 90s 内没有接收到某个微服务实例的心跳，会注销该实例。

固定时间内大量实例被注销，可能会严重威胁整个微服务架构的可用性。为了解决这个问题，Eureka 开发了自我保护机制

> Eureka Server 在运行期间会去统计心跳失败比例在 15 分钟之内是否高于 85%，如果高于 85%，Eureka Server 即会进入自我保护机制。

**Eureka Server 进入自我保护机制，会出现以下几种情况：**

-  Eureka 不再从注册列表中移除因为长时间没收到心跳而应该过期的服务 

-  Eureka 仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用) 

-  当网络稳定时，当前实例新的注册信息会被同步到其它节点中 

Eureka 自我保护机制是为了防止误杀服务而提供的一个机制。当个别客户端出现心跳失联时，则认为是客户端的问题，剔除掉客户端；

* 当 Eureka 捕获到大量的心跳失败时，则认为可能是网络问题，进入自我保护机制；

* 当客户端心跳恢复时，Eureka 会自动退出自我保护机制。

如果在保护期内刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，即会调用失败。

对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。

```properties
eureka.server.enable-self-preservation=true
```

**多级缓存机制**

Eureka Server为了避免同时读写内存数据结构造成的并发冲突问题，还采用了**多级缓存机制**来进一步提升服务请求的响应速度。

在拉取注册表的时候：

-  首先从**ReadOnlyCacheMap**里查缓存的注册表。 

-  若没有，就找**ReadWriteCacheMap**里缓存的注册表。 

-  如果还没有，就从**内存中获取实际的注册表数据。** 

在注册表发生变更的时候：

-  会在内存中更新变更的注册表数据，同时**过期掉ReadWriteCacheMap**。 

-  此过程不会影响ReadOnlyCacheMap提供人家查询注册表。 

-  一段时间内（默认30秒），各服务拉取注册表会直接读ReadOnlyCacheMap 

-  30秒过后，Eureka Server的后台线程发现ReadWriteCacheMap已经清空了，也会清空ReadOnlyCacheMap中的缓存 

-  下次有服务拉取注册表，又会从内存中获取最新的数据了，同时填充各个缓存。 

**多级缓存机制的优点是什么**

-  尽可能保证了内存注册表数据不会出现频繁的读写冲突问题。 

-  并且进一步保证对Eureka Server的大量请求，都是快速从纯内存走，性能极高 