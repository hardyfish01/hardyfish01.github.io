---
title: Servlet
categories: 
- Tomcat架构解析
---

Servlet容器用来加载和管理业务类。HTTP服务器不直接跟业务类打交道，而是把请求交给Servlet容器去处理，Servlet容器会将请求转发到具体的Servlet，如果这个Servlet还没创建，就加载并实例化这个Servlet，然后调用这个Servlet的接口方法。

* 因此Servlet接口其实是**Servlet容器跟具体业务类之间的接口**。

<img src="https://img-blog.csdnimg.cn/84cfa19d741949b9af0debc7beedcf23.png" style="zoom:25%;" />

**Servlet规范**

Servlet接口和Servlet容器这一整套规范叫作Servlet规范。

* Tomcat和Jetty都按照Servlet规范的要求实现了Servlet容器，同时它们也具有HTTP服务器的功能。

作为Java程序员，如果我们要实现新的业务功能，只需要实现一个Servlet，并把它注册到Tomcat（Servlet容器）中，剩下的事情就由Tomcat帮我们处理了。

**Servlet容器**

* 当客户请求某个资源时，HTTP服务器会用一个ServletRequest对象把客户的请求信息封装起来，然后调用Servlet容器的service方法

* Servlet容器拿到请求后，根据请求的URL和Servlet的映射关系，找到相应的Servlet

* 如果Servlet还没有被加载，就用反射机制创建这个Servlet，并调用Servlet的init方法来完成初始化，接着调用Servlet的service方法来处理请求，把ServletResponse对象返回给HTTP服务器，HTTP服务器会把响应发送给客户端。

<img src="https://img-blog.csdnimg.cn/560dfb37233d4f65bc569746b672a232.png" style="zoom:25%;" />

**扩展机制**

**Filter**

* 过滤器，这个接口允许你对请求和响应做一些统一的定制化处理，比如你可以根据请求的频率来限制访问，或者根据国家地区的不同来修改响应内容。

过滤器的工作原理：Web应用部署完成后，Servlet容器需要实例化Filter并把Filter链接成一个FilterChain。当请求进来时，获取第一个Filter并调用doFilter方法，doFilter方法负责调用这个FilterChain中的下一个Filter。

**Listener**

* 监听器，当Web应用在Servlet容器中运行时，Servlet容器内部会不断的发生各种事件，如Web应用的启动和停止、用户请求到达等。 

Servlet容器提供了一些默认的监听器来监听这些事件，当事件发生时，Servlet容器会负责调用监听器的方法。