---
title: IO模型
categories: 
- 编程语言
- Java基础
---

I/O 模型简单的理解：就是 用什么样的通道进行数据的发送和接收，很大程度上决定了程序通信的性能。

**BIO、NIO、AIO**

Java共支持3种网络编程模型/IO模式：BIO、NIO、AIO。

> Java BIO：

同步并阻塞(传统阻塞型)，服务器实现模式为一个连接对应一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销。

<img src="https://img-blog.csdnimg.cn/565f84f40c5b4dc199718694cd9885d6.png" style="zoom:25%;" />

> Java NIO： 

`同步非阻塞`，服务器实现模式为 `一个线程处理多个请求(连接)`，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求就进行处理。

<img src="https://img-blog.csdnimg.cn/6d535d62465d49508439e0a92732a2ac.png" style="zoom:25%;" />

> `Java AIO(NIO.2)`：

`异步非阻塞`，AIO 引入 `异步通道` 的概念，采用了 `Proactor 模式`，简化了程序编写，`有效的请求才启动线程`，它的特点是：先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用。

**BIO、NIO、AIO适用场景分析**

* BIO方式适用于 连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序简单易理解。

* NIO方式适用于 连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等。编程比较复杂，JDK1.4 开始支持。

* AIO方式适用于 连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂

**BIO编程的简单流程**

* 服务器端启动一个ServerSocket；

* 客户端启动Socket对服务器进行通信，默认情况下服务器端需要对每个客户 建立一个线程与之通讯；

* 客户端发出请求后, 先咨询服务器是否有线程响应，如果没有则会等待，或者被拒绝；

* 如果有响应，客户端线程会等待请求结束后，再继续执行。

**BIO 编程通信实例**

实现思路：

- 创建一个线程池；
- 如果有客户端连接，就创建一个线程与之通讯（单独写一个方法）。

```java
public class BIOServer {
    public static void main(String[] args) throws IOException {
        // 思路：
        // 1.创建一个线程池
        // 2.如果有客户端连接，就创建一个线程与之通讯（单独写一个方法）
        ExecutorService threadPool = Executors.newCachedThreadPool();

        ServerSocket serverSocket = new ServerSocket(6666);
        System.out.println("服务器启动了");
        while (true) {
            // 监听，等待客户端连接
            final Socket socket = serverSocket.accept();
            System.out.println("连接到一个客户端");
            // 创建一个线程与之通讯
            threadPool.execute(new Runnable() {
                public void run() {
                    // 可以和客户端通讯
                    handler(socket);
                }
            });
        }
    }
    // 编写一个和客户端通讯你的方法
    public static void handler(Socket socket) {
        byte[] bytes = new byte[1024];
        // 通过socket获取输入流
        try {
            InputStream inputStream = socket.getInputStream();
            // 循环读取客户端发送的数据
            while (true) {
                int read = inputStream.read(bytes);
                if (read != -1) {
                    // 输出客户端发送的数据
                    System.out.println(new String(bytes,0,read));
                } else { // 否则通讯结束
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            System.out.println("关闭和client的连接");
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

}
```

