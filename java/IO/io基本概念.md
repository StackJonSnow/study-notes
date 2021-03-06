## io模型
### io模型涉及的几个基本概念
1. 同步与异步同步和异步关注的是消息通信机制 (synchronous communication/ asynchronous communication)所谓同步，就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。换句话说，就是由*调用者*主动等待这个*调用*的结果。而异步则是相反，*调用*在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用。典型的异步编程模型比如Node.js举个通俗的例子：你打电话问书店老板有没有《分布式系统》这本书，如果是同步通信机制，书店老板会说，你稍等，”我查一下"，然后开始查啊查，等查好了（可能是5秒，也可能是一天）告诉你结果（返回结果）。而异步通信机制，书店老板直接告诉你我查一下啊，查好了打电话给你，然后直接挂电话了（不返回结果）。然后查好了，他会主动打电话给你。在这里老板通过“回电”这种方式来回调。
2. 阻塞与非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态.阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

同步与异步关注的是消息的通信机制，同步是指调用者主动去获取结果，异步是指被调用者通知调用者，调用者是被动的。阻塞与非阻塞关注的是调用者在等待结果时处于的状态，阻塞时指被调用者返回结果之前，当前线程处于挂起状态，非阻塞是指被调用者返回结果之前，调用线程处于活动状态，可以继续处理其他事情。

### 同步阻塞io
传统的 io 如网络 io 就是同步阻塞 io 模型的实现，serversocket 通过 accept() 实现监听客户端的连接，accept() 是阻塞的，直到有客户端连接，这个方法才会返回，一个线程只能处理一个连接，效率较低，只适合那种并发要求比较低的 web 服务器。

### 同步非阻塞io
java 自 1.4 版本开始就提供了 nio ，这是同步非阻塞 io 模型的一种实现，每个线程可以处理多个客户端连接，通过轮询的方式读取客户端已发送到通道中的数据。nio 适合对并发请求要求较高，和对连接数要求较高的 web 服务器。

### 异步非阻塞io
AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作

### ByteBuffer原理
基本变量：
1.capacity
缓冲区的最大容量，在创建时指定，不可被改变
2.limit
缓冲区当前读写的限制位置，超过限制不允许读写，这个值是可以改变的，通过调用 flit() 将 limit 设置为 position，并将 position 设置为0，通过 limit(int i) 也可将 limit 设置为自定义值。
3.position
下一个读或写的位置，每次读或写这个值都会改变。
4.mark
标记当前 position 的位置，通过调用 mark() 来实现，调用 reset() 会将 position 的值修改为 mark 标记的值。

