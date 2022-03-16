---
title: Feign
categories: 
- SpringCloud微服务实战
---

Feign 是一个声明式的 REST 客户端，它的目的就是让 REST 调用更加简单。

Feign 提供了 HTTP 请求的模板，通过编写简单的接口和插入注解，就可以定义好 HTTP 请求的参数、格式、地址等信息。

而且 Feign 会完全代理 HTTP 请求，我们只需要像调用方法一样调用它就可以完成服务请求及相关处理。

* Spring Cloud 对 Feign 进行了封装，使其支持 SpringMVC 标准注解和 HttpMessageConverters。

* Feign 可以与 Eureka 和 Ribbon 组合使用以支持负载均衡，与 Hystrix 组合使用，支持熔断回退。

如果你没有使用 Spring Cloud，那么可以直接用原生的 Feign 来调用 API，如果你使用了 Spring Cloud，可以直接用 Spring Cloud OpenFeign 来调用 API。

**Feign执行过程**

在使用 feign 时，我们会定义对应的接口类，在接口类上使用 Feign 自带的注解来标识 HTTP 的请求参数信息，当调用接口对应的方法时，Feign 内部会基于面向接口的动态代理方式生成实现类，将请求调用委托到动态代理实现类，负责动态代理的组件是 InvocationHandlerFactory。

* 根据 Contract 规则，解析接口类的注解信息，翻译成 Feign 内部能识别的信息。

* Feign 默认有一套自己的协议规范，我们也可以自定义其他的规范来进行扩展，在 Spring Cloud OpenFeign中就扩展了 SpringMvcContract，这样做的目的是为了降低学习和使用成本，客户端和服务端使用同一个接口定义，发布成 SDK 给调用方使用。

* MethodHandler 在执行的时候会生成 Request 对象，在构建 Request 对象的时候会为其设置拦截器，交由 Client 执行前记录一些日志，Client 执行完成后也记录一些日志，然后使 Decoder 进行相应结果的解码操作，并返回结果。

<img src="https://img-blog.csdnimg.cn/f109761e05c94186bfe4b6bff749ca98.png" style="zoom:50%;" />