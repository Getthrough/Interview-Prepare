# Interview-Prepare
Record something which are benifit to the interview

## Netty 一个基于 Java NIO 的高性能开发框架。

Java 网络编程简史

说到 NIO，就离不开网络编程。传统的 Java 网络编程是基于同步阻塞的 BIO 流进行 socket 编程，这样种方式有一个优点就是编程简单，
但同样缺点也很明显：对于每个一个客户端的请求都需要创建一条线程去处理 IO 任务。每一条线程本身就会占用一定的 JVM 内存资源，并且在线程数量超过一定阈值后，
由于频繁的上下文切换，性能不升反降。此乃第一个阶段。

对于处理创建过多线程这个问题，其解决方式都是大同小异的，那就是应用“池”的模式（类比）。
一个池，它维护了一个任务队列和指定数量的若干个活跃线程，当服务端接受到客户端的请求时，将该请求封装成一个任务（实现 Runnable 接口）放入池中的任务队列，
由池中的线程进行轮流消费。这样限定处理线程的数量，就解决了当客户端请求大幅增长时，服务器里宕机的风险。这种模式对于“每个请求对应一条线程”而言，
的确是一种改良，它被称为“伪异步 IO”。此乃第二个阶段
那么它存在什么问题呢，看上去不是挺优秀的？
设想下，当客户端网络较差或者发送的数据量较大时，服务端读取时是处于较长时间的阻塞状态；同样，当客户端网络条件较差时，从 TCP 缓冲区取数据缓慢，
导致一段时间后缓冲区被占满，服务端将会在写出时阻塞。此时任务队列中任务会越积越多，直到无法接受任务，此时，客户端将发生连接超时...

直到... NIO 的出现。

在 BIO 的套接字（socket）编程中，有两个产生 socket 的类，它们分别是 “Socket” 和 “ServerSocket”，
在 NIO 中，提供了 “SocketChannel” 和 “ServerSocketChannel” 两种不同的套接字实现。这两种通道同时支持阻塞和非阻塞模式。

【Buffer】

BIO 面向流（Stream）编程，在 NIO 中是面向缓冲区（Buffer）的。
除了 boolean 类型，其余所有 Java 基本数据类型都对应着一个缓冲区，ByteBuffer，IntBuffer，LongBuffer等。

【Channel】

网络数据通过 Channel 通道进行读取和写入，读取和写入操作可以同时进行，也就是说 Channel 是全双工的。这一点与流不同，流是单向的（输入流、输出流）。

【Selector】

在《Netty权威指南》（李林峰著）一书中，作者将其称为“多路复用器”，这是因为一条 Reactor 线程会初始化 Selector，对所有注册在 Selector 上的 Channel 进行轮询，并通过 SelectionKey 能获取所有就绪的 Channel 集合，于是一条线程即可接入大量的客户端，因此将其称之为“多路复用器”。

NIO 服务端开发流程：

1. 打开 ServerSocketChannel 管道，它是所有客户端连接的父管道；
2. 绑定监听端口，设置连接为非阻塞模式（configureBlocking）及还有其他若干设置；
3. 创建 Reactor 线程，初始化 Selector，启动线程；
4. 将 ServerSocketChannel 注册到 Selector 上，并监听 OP_ACCEPT 事件；
5. 轮询 SelectionKey；
6. 监听并处理客户端接入请求，建立物理链路；
7. 设置客户端链路为非阻塞模式；
8. 将处理客户端请求的 Channel 注册到 Selector 上，监听 OP_READ 事件（读取客户端发送的数据）；
9. 异步将客户端数据读取到缓冲区；
10. 对消息数据解码，进行业务操作；
11. 将业务操作后封装的 POJO 对象编码成 ByteBuffer（此 Buffer 大多用来处理网络操作），并异步写出。

NIO 客户端开发流程：

1. 打开 SocketChannel 管道；
2. 绑定客户端地址，设置连接为非阻塞模式和 TCP 连接参数；
3. 连接服务器（返回boolean）；
4. 判断是否连接成功，如果注册成功则向 Channel 注册 OP_READ，否则注册 OP_CONNECT；
5. 创建 Reactor 线程和 Selector，启动线程；
6. 判断 channel 是否连接成功，如果连接成功，注册 OP_READ 事件到 Selector 上，并写出（write）消息；否则注册 OP_CONNECT；
7. 在 run() 方法中轮询已经就绪的 key，判断 key 处于可读状态，若可读，则读取数据到 ByteBuffer 进行处理；
8. 结束后释放资源（Selector）。

使用 NIO 非阻塞模式开发，还需要注意“写半包”问题，因为异步的操作不能保证一次性将所有数据发送完毕。
由此可见 NIO 编程的复杂性相较于传统 IO 来说，增加了很多。

NIO 2.0
























