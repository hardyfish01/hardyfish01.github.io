---
title: NIO
categories: 
- 编程语言
- Java基础
---

**基本概念**

从 JDK1.4 开始，Java 提供了一系列改进的 输入/输出 的新特性，被统称为 NIO(即 New IO)，是 同步非阻塞 的。

NIO 相关类都被放在 `java.nio` 包及子包下，并且对原` java.io` 包中的很多类进行改写。

> NIO 有三大核心部分：Channel(通道)、Buffer(缓冲区)、Selector(选择器)

NIO是 面向缓冲区 ，或者面向 块 编程的（因为有了 `Buffer`）。

数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性。

**Java NIO的非阻塞模式：**

使一个线程从某个 通道 发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 

非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。

**Selector（选择器）：**

获取到通道的事件，就调用 thread 会去处理；如果这个通道没有事件了，thread 就去处理别的通道的事件；如果所有通道都没有事件了，thread 还可以去其它任务。反正thread没有阻塞。

**通俗理解：**

* NIO是可以做到用一个线程来处理多个操作的。假设有10000个请求过来，根据实际情况，可以分配50或者100个线程来处理。不像之前的阻塞IO那样，非得分配10000个。

> HTTP2.0 使用了 多路复用 的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.1大了好几个数量级。

**NIO 与 BIO 比较**

* BIO 以 流 的方式处理数据，而 NIO 以 块 的方式处理数据，块 I/O 的效率比流 I/O 高很多；

* BIO 是 阻塞 的，NIO 则是 非阻塞 的

* BIO基于 字节流和字符流 进行操作，而 NIO 基于 `Channel`(通道) 和 Buffer(缓冲区) 进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。

* Selector(选择器)用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道。

**NIO 三大核心组件之间的关系**

<img src="https://img-blog.csdnimg.cn/7813c3b3a85e44c88a10a957080915fb.png" style="zoom:25%;" />

* 每个channel 都会对应一个Buffer；

* Selector 对应一个线程， 一个线程对应多个channel(连接)；

* 该图反应了有三个channel 注册到 Selector；

* 程序切换到哪个channel 是由 事件（Event） 决定的，Event 就是一个重要的概念；

* Selector 会根据不同的事件，在各个通道上切换；

* Buffer 就是一个 内存块 ， 底层是有一个数组；

* 数据的 读取/写入 是通过 Buffer， 这个和BIO不同 , BIO 中 要么是输入流，要么是输出流，不能双向，但是NIO的Buffer 是可以读也可以写，但是需要 flip 方法 切换 读/写 状态；

* channel 是 双向 的，可以返回底层操作系统的情况，比如Linux 底层的操作系统通道就是双向的。

# Buffer

缓冲区本质上是一个可以 读写 数据的 内存块，可以理解成是一个容器对象(含数组)，该对象提供了一组方法，可以更轻松地使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。

Channel 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer

在 NIO 中，Buffer 是一个顶层父类，它是一个 **抽象类**

**Buffer类中的重要属性**

```java
private int mark = -1;
private int position = 0;
private int limit;
private int capacity;
```

* Capacity：容量，即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变

* Limit：表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作。且极限是可以修改

* Position：位置，下一个要被读或写的元素的索引，每次读写缓冲区数据时都会改变改值，为下次读写作准备

* Mark：标记

**Buffer的基本使用示例**

```java
public class BasicBuffer {
    public static void main(String[] args) {
        // 举例说明 Buffer 的使用
        // 创建一个 Buffer，大小为 6，可以存放 6 个 int
        IntBuffer intBuffer = IntBuffer.allocate(5);

        // 向 Buffer 中存放数据
        for (int i = 0; i < intBuffer.capacity(); i++) {
            intBuffer.put(i * 2);
        }

        // 从 Buffer 中读取数据
        // 将 Buffer 转换，读/写 切换 !!!
        intBuffer.flip();

        while (intBuffer.hasRemaining()) {
            System.out.println(intBuffer.get());
        }
    }
}
```

# Channel

NIO 的通道类似于流，但有些区别如下：

- 通道可以 `同时进行读写`，而流只能读或者只能写；
- 通道可以实现 `异步读写数据`；
- 通道可以 `从缓冲读数据`，也可以 `写数据到缓冲`。

Channel在NIO中是一个接口

```java
public interface Channel extends Closeable{}
```

常用的 Channel 类有：FileChannel、DatagramChannel、ServerSocketChannel 和SocketChannel。

* FileChannel 用于 文件 的数据读写

* DatagramChannel 用于 UDP 的数据读写

* ServerSocketChannel 和 SocketChannel 用于 TCP 的数据读写。

  * ServerSocketChannel 类似ServerSocket；

  * SocketChannel 类似 Socket。

**ServerSocketChannel 和 SocketChannel的区别**

我们首先会在服务器端创建一个ServelSocketChannel，它真实的类型是ServelSocketChannelImpl，当有一个连接来连接Server时，由ServelSocketChannel生成一个与该客户端对应的一个通道，这个通道的类型就是SocketChannel，它的真实类型是SocketChannelImpl，客户端通过这个通道与Server进行通讯。

**MappedByteBuffer - 直接在堆外内存中修改文件**

`MappedByteBuffer`， 可以让文件直接在内存（`堆外的内存`）中进行修改，操作系统不需要再拷贝一份，而如何同步到文件由NIO 来完成。

```java
public class NIOMappedByteBuffer {
    public static void main(String[] args) throws Exception {
        RandomAccessFile randomAccessFile = new RandomAccessFile("d:\\file01.txt", "rw");
        FileChannel channel = randomAccessFile.getChannel();
        /*
        参数1：FileChannel.MapMode.READ_WRITE，使用的是读写位置
        参数2：0，可以直接修改的起始位置
        参数3：5，映射到内存的大小，即将 file01.txt 的多少个字节映射到内存。
                 可以直接修改的范围就是 0 - 5。
         */
        // MappedByteBuffer是一个抽象类，实际类型是 DirectByteBuffer
        MappedByteBuffer mappedByteBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 5);

        // 将第一个位置的值修改为 H
        mappedByteBuffer.put(0, (byte) 'H');

        randomAccessFile.close();
    }

}
```

**应用实例1 - 本地文件写数据**

示例要求:

- 使用前面学习后的ByteBuffer(缓冲) 和 FileChannel(通道)， 将 “hello” 写入到`file01.txt` 中。
- 文件不存在就创建。

```java
public class NIOFileChannel {
    public static void main(String[] args) throws IOException {
        String str = "hello world";

        // 创建一个输出流
        FileOutputStream fileOutputStream = new FileOutputStream("d:\\file01.txt");

        // 通过fileOutputStream 获取对应的 FileChannel
        // 注意：是 fileOutputStream 中包裹了 FileChannel
        FileChannel channel = fileOutputStream.getChannel();

        // 创建Buffer(缓冲区)
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        // 将 str 放入 byteBuffer
        byteBuffer.put(str.getBytes());

        // 切换 byteBuffer 为 write 模式
        byteBuffer.flip();

        // 将 byteBuffer 中的数据写入到 fileChannel
        channel.write(byteBuffer);

        fileOutputStream.close();
    }
}
```

**应用实例2 - 本地文件读数据**

实例要求:

- 使用 ByteBuffer(缓冲) 和 FileChannel(通道)， 将 `file01.txt` 中的数据读入到程序，并显示在控制台屏幕；
- 假定文件已经存在。

```java
public class NIOFileChannel02 {
    public static void main(String[] args) throws IOException {
        String str = "hello world";

        // 创建一个输入流
        File file = new File("d:\\file01.txt");
        FileInputStream fileInputStream = new FileInputStream(file);

        // 通过fileOutputStream 获取对应的 FileChannel
        FileChannel channel = fileInputStream.getChannel();

        // 创建Buffer(缓冲区)
        ByteBuffer byteBuffer = ByteBuffer.allocate((int) file.length());

        // 将 fileChannel 中的数据读入到 byteBuffer
        channel.read(byteBuffer);

        // 将 byteBuffer 中的数据转成 String
        System.out.println(new String(byteBuffer.array()));

        fileInputStream.close();
    }
}
```

**应用实例3 - 本地文件的拷贝**

实例要求:

- 使用 ByteBuffer(缓冲) 和 FileChannel(通道)，完成文件的拷贝
- 拷贝一个文本文件 `file01.txt` , 放在项目下即可

```java
public class NIOFileChannel03 {
    public static void main(String[] args) throws IOException {
        String str = "hello world";

        // 创建一个输入流
        File file = new File("d:\\file01.txt");
        FileInputStream fileInputStream = new FileInputStream(file);

        // 创建一个输出流
        FileOutputStream fileOutputStream = new FileOutputStream("d:\\file02.txt");

        // 通过 fileInputStream 获取对应的 FileChannel
        FileChannel channel01 = fileInputStream.getChannel();

        // 通过 fileOutputStream 获取对应的 FileChannel
        FileChannel channel02 = fileOutputStream.getChannel();


        // 创建Buffer(缓冲区)
        ByteBuffer byteBuffer = ByteBuffer.allocate(512);

        while (true) {
            // 一次可能读不完，所以清空 byteBuffer
            byteBuffer.clear();

            int read = channel01.read(byteBuffer);
            if (read == -1) {
                break;
            }
            byteBuffer.flip();
            channel02.write(byteBuffer);
        }

        fileInputStream.close();
        fileOutputStream.close();
    }
}
```

**应用实例4 - 拷贝文件（transferFrom 方法）**

实例要求:

- 使用 FileChannel(通道) 和 方法 transferFrom ，完成文件的拷贝；
- 拷贝一张图片。

```java
public class NIOFileChannel04 {
    public static void main(String[] args) throws IOException {
        FileInputStream fileInputStream = new FileInputStream("d:\\a.jpg");
        FileOutputStream fileOutputStream = new FileOutputStream("d:\\a2.jpg");

        FileChannel source = fileInputStream.getChannel();
        FileChannel target = fileOutputStream.getChannel();

        // 使用 transferForm 完成拷贝
        target.transferFrom(source,0,source.size());

        source.close();
        target.close();
        fileInputStream.close();
        fileOutputStream.close();
    }
}
```

# Selector

**基本介绍**

* Java 的 NIO，用非阻塞的 IO 方式。可以用一个线程，处理多个的客户端连接，就会使用到 `Selector`(选择器)；

* Selector 能够检测多个注册的通道上是否有事件发生(注意:多个`Channel`以事件的方式可以注册到同一个`Selector`)，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就 可以只用一个单线程去管理多个通道，也就是管理多个连接和请求；

* 只有在 连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程；

* 避免了多线程之间的上下文切换导致的开销。

**Selector作用详细说明**

* Netty 的 IO 线程 NioEventLoop 聚合了 `Selector`(选择器，也叫多路复用器)，可以同时并发处理成百上千个客户端连接。

* 当线程从某客户端 Socket 通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。

* 线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入和输出通道。

* 由于读写操作都是 非阻塞 的，这就可以充分提升 IO线程的运行效率，避免由于频繁 I/O 阻塞导致的线程挂起。

* 一个 I/O 线程可以并发处理 N 个客户端连接和读写操作，这从根本上解决了传统同步阻塞 I/O 一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。

**Selector 类**

```java
public abstract class Selector implements Closeable {
	// 得到一个选择器对象
	public static Selector open();
	// 监控所有注册的通道，当其中有 IO 操作可以进行时，将对应的 SelectionKey 
	// 加入到内部集合中并返回，参数用来设置超时时间
	public int select(long timeout);
	// 从内部集合中得到所有的 SelectionKey
	public Set<SelectionKey> selectedKeys();
	// 唤醒selector
	selector.wakeup();
	// 不阻塞，立马返还
	selector.selectNow();
}
```

**SelectionKey 作用：**

* Selector 对象调用 select() 方法会返回一个 SelectionKey 集合，根据 SelectionKey 获取到对应的channel，然后处理channel中发生的事件。

`select() 或 select(long timeout)`

* `select()` ：调用它会一直阻塞，直到获取注册到的Selector中的channel至少有一个channel发生它所关心的事件才返回，返回的是发生事件的channel的SelectionKey。

* `select(long timeout)`：指定阻塞事件，到时见即使没有监听到任何事件也会返回。

**SelectionKey**

SelectionKey 表示 Selector 和 网络通道 的 注册关系, 共四种:

* int OP_ACCEPT：有新的网络连接可以 accept，值为 16；

* int OP_CONNECT：代表连接已经建立，值为 8；

* int OP_READ：代表读操作，值为 1；

* int OP_WRITE：代表写操作，值为 4。

**Selector、SelectionKey、ServerScoketChannel、SocketChannel关系梳理**

<img src="https://img-blog.csdnimg.cn/83adec5a59ab4e2da59cdfe919e86860.png" style="zoom:25%;" />

* 当客户端连接时，会通过 ServerSocketChannel （ServerSocketChannel也需要注册到selector上）得到 SocketChannel；

* Selector 进行监听 ，通过 select() 方法，返回有事件发生的通道的个数；

* socketChannel调用 `register(Selector sel, int ops)` 方法，注册到Selector上，一个selector上可以注册多个SocketChannel；

* socketChannel注册成功后会返回一个 SelectionKey，用于和该Selector 关联，多个socketChannel注册成功后就会有一个 SelectionKey 集合；

* Selector 通过 select() 方法，返回有事件发生的channel的个数；

* 进一步得到各个 SelectionKey (有事件发生的channel的SelectionKey )；

* 再通过 SelectionKey 反向获取 SocketChannel （通过 channel() 方法）；

* 可以通过 得到的 channel，完成业务处理。

# 入门案例

**服务器端代码**

```java
public class NIOServer {

    public static void main(String[] args) throws IOException {

        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        // 获取 Selector 对象
        Selector selector = Selector.open();

        // 绑定端口 6666 在服务端监听
        serverSocketChannel.socket().bind(new InetSocketAddress(6666));
        // 设置为非阻塞
        serverSocketChannel.configureBlocking(false);

        // 注意：ServerSocketChannel 也需要注册到 Selector 上
        // 因为客户端总得需要一个通道来通知Server，然后Server再分配一个和cient的单独通道
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        // 循环等待客户端连接
        while (true) {
            // 等待1秒，如果没有事件发生，返回
            if (selector.select(1000) == 0) {
                System.out.println("服务器等待了1秒，客户端无连接");
                continue;
            }
            // 获取发生关注事件的channel的SelectionKey集合
            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            Iterator<SelectionKey> keyIterator = selectionKeys.iterator();

            while (keyIterator.hasNext()) {
                // 获取 SelectionKey
                SelectionKey key = keyIterator.next();
                // 根据 SelectionKey 对应的 channel 发生的事件做相应的处理
                if (key.isAcceptable()) { // 如果是 OP_ACCEPT，代表有新的客户端连接Server
                    // 分配一个 SocketChannel 给客户端
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    System.out.println("客户端连接成功，生成了一个socketChannel "+socketChannel.hashCode());
                    // 将 socketChannel 设置为 非阻塞
                    socketChannel.configureBlocking(false);
                    // 将这个 SocketChannel 注册到 selector，关注事件为 OP_READ，
                    // 同时关联一个Buffer
                    socketChannel.register(selector,SelectionKey.OP_READ, ByteBuffer.allocate(1024));
                }

                if (key.isReadable()) {// 发生 事件为 OP_READ
                    // 通过 key 反向获取到 channel
                    SocketChannel socketChannel = (SocketChannel) key.channel();
                    // 获取到该channel关联的buffer
                    ByteBuffer buffer = (ByteBuffer) key.attachment();
                    // 将 channel 中的数据读入到 Buffer 中
                    socketChannel.read(buffer);
                    System.out.println("客户端的数据："+new String(buffer.array()));

                }
                // 手动从集合中移除当前的 selectionKey，防止重复操作
                keyIterator.remove();

            }

        }

    }
}
```

**客户端代码**

```java
public class NIOClient {
    public static void main(String[] args) throws IOException {
        // 得到一个网络通道
        SocketChannel socketChannel = SocketChannel.open();
        // 设置非阻塞模式
        socketChannel.configureBlocking(false);
        // 提供服务器端的 ip 和 port
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 6666);
        // 连接服务器，非阻塞式连接
        if (!socketChannel.connect(inetSocketAddress)) {
            while (!socketChannel.finishConnect()) {
                System.out.println("因为连接需要时间，客户端不会阻塞，可以做其它工作");
            }
        }
        // 如果连接成功就发送数据
        String str = "hello world";
        // 根据字节数的大小，自动设置buffer的大小
        ByteBuffer byteBuffer = ByteBuffer.wrap(str.getBytes());
        // 发送数据，将buffer中的数据写入到channel
        socketChannel.write(byteBuffer);
        System.in.read();

    }
}
```

