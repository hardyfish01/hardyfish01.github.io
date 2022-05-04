---
title: Future回调机制
categories: 
- 中间件
- Netty
---

Java的Future大家应该比较清楚，以一种非阻塞的方式，快速返回

但是这种方式一个比较大的缺点是用户必须通过`.get()`方式来获取结果，无法精确了解完成时间。

Netty扩展了Java的Future，最主要的改进就是增加了监听器Listener接口，通过监听器可以让异步执行更加有效率，不需要通过get来等待异步执行结束，而是通过监听器回调来精确地控制异步执行结束的时间点。

ChannelFuture接口扩展了Netty的Future接口，表示一种没有返回值的异步调用，同时关联了Channel，跟一个Channel绑定。

**常见有如下操作：**

* 通过 isDone 方法来判断当前操作是否完成；

* 通过 isSuccess 方法来判断已完成的当前操作是否成功；

* 通过 getCause 方法来获取已完成的当前操作失败的原因；

* 通过 isCancelled 方法来判断已完成的当前操作是否被取消；

* 通过 addListener 方法来注册监听器，当操作已完成(isDone 方法返回完成)，将会通知 指定的监听器；如果 Future 对象已完成，则通知指定的监听器

代码示例：

```java
private void doConnect(final Logger logger,final String host, final int port) {
    ChannelFuture future = bootstrap.connect(new InetSocketAddress(host, port));

    future.addListener(new ChannelFutureListener() {
        public void operationComplete(ChannelFuture f) throws Exception {
            if (!f.isSuccess()) {
                logger.info("Started Tcp Client Failed");
                f.channel().eventLoop().schedule( new Runnable() {
                    @Override
                    public void run() {
                        doSomeThing();
                    }
                }, 200, TimeUnit.MILLISECONDS);
            }
        }
    });
}
```

