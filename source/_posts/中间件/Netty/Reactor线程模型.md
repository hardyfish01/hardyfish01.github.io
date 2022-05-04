---
title: Reactor线程模型
categories: 
- 中间件
- Netty
---

Reactor是反应堆的意思，Reactor模型，是指通过一个或多个输入同时传递给服务处理器的服务请求的事件驱动处理模式。 

服务端程序处理传入多路请求，并将它们同步分派给请求对应的处理线程，Reactor模式也叫Dispatcher模式，即I/O多了复用统一监听事件，收到事件后分发(Dispatch给某进程)。

**Reactor 的线程模型有三种:**

- 单线程模型
- 多线程模型
- 主从多线程模型

**单线程模型**

<img src="https://img-blog.csdnimg.cn/d24584be79a5453e9c61752881bd760e.png" style="zoom:25%;" />

所谓单线程，即 acceptor 处理和 handler 处理都在一个线程中处理

当其中某个 handler 阻塞时，会导致其他所有的 client 的 handler 都得不到执行，并且更严重的是，handler 的阻塞也会导致整个服务不能接收新的 client 请求(因为 acceptor 也被阻塞了)

**多线程模型**

Reactor 的多线程模型与单线程模型的区别就是 acceptor 是一个单独的线程处理，并且有一组特定的 NIO 线程来负责各个客户端连接的 IO 操作

<img src="https://img-blog.csdnimg.cn/ebaff226cfbd4d639b0a3fb0505e57d7.png" style="zoom:25%;" />

Reactor 多线程模型 有如下特点:

- 有专门一个线程，即 Acceptor 线程用于监听客户端的TCP连接请求。
- 客户端连接的 IO 操作都是由一个特定的 NIO 线程池负责，每个客户端连接都与一个特定的 NIO 线程绑定，因此在这个客户端连接中的所有 IO 操作都是在同一个线程中完成的。
- 客户端连接有很多，但是 NIO 线程数是比较少的。因此一个 NIO 线程可以同时绑定到多个客户端连接中。

**主从多线程模型**

如果我们的服务器需要同时处理大量的客户端连接请求或我们需要在客户端连接时，进行一些权限的检查，那么单线程的 Acceptor 很有可能就处理不过来，造成了大量的客户端不能连接到服务器。

Reactor 的主从多线程模型就是在这样的情况下提出来的，它的特点是: 服务器端接收客户端的连接请求不再是一个线程，而是由一个独立的线程池组成

<img src="https://img-blog.csdnimg.cn/33778a1b3da442309a007b23bc2cd6f5.png" style="zoom:25%;" />

这种模型在许多项目中广泛使用，包括 Nginx 主从 Reactor 多进程模型， Memcached 主从多线程，Netty 主从多线程模型的支持

**3种模式用生活案例来理解：**

> 1. 单 Reactor 单线程，前台接待员和服务员是同一个人，全程为顾客服务
> 2. 单 Reactor 多线程，1 个前台接待员，多个服务员，接待员只负责接待
> 3. 主从 Reactor 多线程，多个前台接待员，多个服务生

**Netty架构模型**

Netty是一个异步网络通信框架，异步主要体现在对java Future的拓展，基于Future/Listener的回调机制完成了对事件的监听

通过Channel完成了对数据的传输，使用NioEventLoop工作线程，通过执行ChanelHandler完成了对ChannelPipeline上的数据处理。

<img src="https://img-blog.csdnimg.cn/90f02fd8f6004cc587df7bfe3cf4e72f.png" style="zoom:25%;" />