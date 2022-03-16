---
title: Zuul
categories: 
- SpringCloud微服务实战
---

在 Spring Cloud 中，针对 API 网关的实现提供了两种解决方案。

* 一种是集成 Netflix 中的 Zuul 网关，一种是自研 Spring Cloud Gateway。

Spring Cloud Netflix Zuul 同样是基于 Netflix 旗下的 Zuul 组件。

* Zuul 也是 Netflix OSS 中的一员，是一个基于 JVM 路由和服务端的负载均衡器。

* 提供了路由、监控、弹性、安全等服务。Zuul 能够与 Eureka、Ribbon、Hystrix 等组件配合使用。

**过滤器**

过滤器是 Zuul 中最核心的内容，过滤器可以对请求或响应结果进行处理，Zuul 还支持动态加载、编译、运行这些过滤器，过滤器的使用方式是采取责任链的方式进行处理，过滤器之间通过 RequestContext 来传递上下文，通过过滤器可以扩展很多高级功能。

Zuul 中的过滤器总共有 4 种类型，每种类型都有对应的使用场景：

* pre 过滤器：可以在请求被路由之前调用。适用于身份认证的场景，认证通过后再继续执行下面的流程。

* route 过滤器：在路由请求时被调用。适用于灰度发布的场景，在将要路由的时候可以做一些自定义的逻辑。

* post 过滤器：在 route 和 error 过滤器之后被调用。这种过滤器将请求路由到达具体的服务之后执行。适用于添加响应头，记录响应日志等应用场景。

* error 过滤器：处理请求发生错误时被调用。在执行过程中发送错误时会进入 error 过滤器，可以用来统一记录错误信息。

<img src="https://img-blog.csdnimg.cn/49d043d31a6d428f8a9396cd9ab0a4bf.png" style="zoom:25%;" />

**请求生命周期**

当一个请求进来时，会先进入 pre 过滤器，在 pre 过滤器执行完后，接着就到了 routing 过滤器中，开始路由到具体的服务中，路由完成后，接着就到了 post 过滤器中，然后将请求结果返回给客户端。

如果在这个过程中出现异常，则会进入 error 过滤器中，这就是请求在整个 Zuul 中的生命周期。

<img src="https://img-blog.csdnimg.cn/36eda8ae82864b0ab20556f25d1fb9e5.png" style="zoom:25%;" />

对应的源码在 ZuulServlet 中，我们可以打开 ZuulServlet 的源码：

* service 方法中就是执行过滤器的逻辑，首先是 preRoute 方法，也就是执行 pre 过滤器，如果异常了就会执行 error 过滤器和 post 过滤器，接着就是 routing 过滤器，这就是整个过滤器执行流程对应的源码部分。

**Zuul 路由**

对于 Zuul 的路由方式，我总结了四种，每种方式都有它的使用场景，下面来介绍下这四种方式：

* 第一种路由方式是 URL 路由，不具备负载均衡。这里的不具备负载均衡是指 Zuul 中转发请求的时候不具备负载均衡，但如果配置的是域名的话，就相当于集中式的负载均衡了。

* 第二种路由方式是配置多个 URL 负载均衡，本质上还是使用了 Ribbon 来进行负载均衡，所以我们需要通过配置 Ribbon 的 servers 来做负载。将配置的 URL 改成 serviceId，serviceId 也就是 Ribbon 的名称，通过` serviceId.ribbon.listOfServers `配置多个 URL 地址，这样 URL 路由也具备了客户端负载均衡的能力了。

* 第三种路由方式是为所有请求加前缀，这种方式适合需要对 API 有一个统一的前缀的场景，比如我们想统一的前缀为 openapi，那么就需要增加` zuul.prefix=/openapi`，这样在访问时，请求地址也需要加上 openapi，比如之前的是` cxytiandi.com/blogs`，添加完前缀就变成了`cxytiandi.com/openapi/blogs`。

* 第四种方式是基于 Eureka 代理服务，我们使用网关，90% 的需求都是转发内部服务，这些服务都会注册到 Eureka 中，虽然可以手动配置 Ribbon 的 servers 列表，但这种方式需要人为干预。所以我们需要在 Zuul 中集成 Eureka，在路由转发时可以转发到 Eureka 中注册的服务上，这样就很方便了，不需要我们去关心服务的上下线。

**Zuul 容错与回退**

Spring Cloud 中，Zuul 默认整合了 Hystrix，当后端服务异常时可以为 Zuul 添加回退功能，返回默认的数据给客户端。

我们需要在 Zuul 中实现 FallbackProvider 这个类来实现回退逻辑。

* 需要实现 FallbackProvider 中的 getRoute 方法，告诉 Zuul 它是负责哪个路由的熔断，如果想全局进行处理可以返回` * `号表示所有。

* 而 fallbackResponse 方法则是告诉 Zuul 断路出现时，需要返回给客户端什么数据。

* 可以指定 HttpStatus、HttpHeaders 等信息，最重要的就是返回的数据在 getBody 中进行设置。

Zuul 中默认采用信号量隔离机制，如果想要换成线程，需要配置` zuul.ribbon-isolation-strategy=THREAD`，配置后所有的路由对应的 Command 都在一个线程池中执行，这样其实达不到隔离的效果，所以我们需要增加一个` zuul.thread-pool.use-separate-thread-pools `的配置，让每个路由都使用独立的线程池，`zuul.thread-pool.thread-pool-key-prefix `可以为线程池配置对应的前缀，方便调试。