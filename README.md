# Netty Reading Project（Netty源代码阅读项目）

Unix下的IO模型

IO调用会涉及到用户空间和内核空间之间的交互。发送数据时，应用程序会将数据从用户空间拷贝到内核空间，再由内核执行发送动作。当接收数据时，则需要等内核将数据准备好之后再从内核空间拷贝到用户空间。

1. 阻塞式I/O：这是最简单和古老的模式。以接收数据为例。应用的接收数据操作会等待内核准备好数据并复制到用户空间后再返回。在此期间，接收数据的操作是被阻塞住的。
2. 非阻塞式I/O：仍然以接收数据为例，应用接收数据的操作发现内核没有准备数据则直接返回错误，意味着应用需要轮询。等到内核准备好数据后，应用会等数据拷贝到用户空间再返回。准确说，第一个阶段是非阻塞的，而第二个阶段还是阻塞的。
3. I/O复用：应用程序阻塞在select或poll上而不是真正的I/O系统调用上。当应用关心的事件发生时，应用再调用内核将数据从内核拷贝到用户空间，最后返回。
4. 信号驱动I/O：比较像非阻塞式I/O，只是在套接字上注册了信号调用方法，注册完之后就返回。当内核准备好数据后，就会给进程发送信号。进程就可以读取数据了。
5. 异步I/O：通过系统调用，向内核注册异步回调函数。当内核收到数据，并将数据从内存复制到用户空间后，再依据注册的方法通知进程。与信号I/O的最大差别是，信号I/O是通知应用数据已经可以读取了，而异步I/O是通知应用程序数据已经读取完毕了。

前四种模型主要区别在第一阶段，第二阶段都是一样的，数据从内核复制到用户空间期间，进程阻塞在recvfrom调用。第五种异步I/O在这两个阶段都要处理。
POSIX定义同步I/O和异步I/O。
同步I/O导致请求进程阻塞，直到I/O完成。
异步I/O不导致请求进程阻塞。从这个角度说，阻塞式I/O、非阻塞式I/O、I/O复用和信号驱动式都是同步I/O模型，因为其中真正的I/O操作recvfrom将阻塞进程。只有异步I/O模型与POSIX定义的I/O相匹配。

在 NIO 中应用多线程，主要有三种思路：

思路一：只有一个选择器，把选择键的处理动作以任务的形式投放到线程池中处理。

思路二：多个选择器，每一个选择器搭配一个线程，并且在自身的线程中死循环等待事件发生以及处理事件。

思路三：思路二的变种，区分 2 组选择器，一组选择器专门用于服务端通道，用于客户端链接的接入；一组选择器专门用于客户端通道，用于客户端通道的数据读写。

Java在1.7版本中为我们带来了异步IO的支持。

Netty is an asynchronous event-driven network application framework for rapid development of maintainable high performance protocol servers & clients.

## Links

* [Web Site](https://netty.io/)
* [Downloads](https://netty.io/downloads.html)
* [Documentation](https://netty.io/wiki/)
* [@netty_project](https://twitter.com/netty_project)

## How to build

For the detailed information about building and developing Netty, please visit [the developer guide](https://netty.io/wiki/developer-guide.html).  This page only gives very basic information.

You require the following to build Netty:

* Latest stable [Oracle JDK 7](http://www.oracle.com/technetwork/java/)
* Latest stable [Apache Maven](http://maven.apache.org/)
* If you are on Linux, you need [additional development packages](https://netty.io/wiki/native-transports.html) installed on your system, because you'll build the native transport.

Note that this is build-time requirement.  JDK 5 (for 3.x) or 6 (for 4.0+) is enough to run your Netty-based application.

## Branches to look

Development of all versions takes place in each branch whose name is identical to `<majorVersion>.<minorVersion>`.  For example, the development of 3.9 and 4.0 resides in [the branch '3.9'](https://github.com/netty/netty/tree/3.9) and [the branch '4.0'](https://github.com/netty/netty/tree/4.0) respectively.

## Usage with JDK 9

Netty can be used in modular JDK9 applications as a collection of automatic modules. The module names follow the
reverse-DNS style, and are derived from subproject names rather than root packages due to historical reasons. They
are listed below:

 * `io.netty.all`
 * `io.netty.buffer`
 * `io.netty.codec`
 * `io.netty.codec.dns`
 * `io.netty.codec.haproxy`
 * `io.netty.codec.http`
 * `io.netty.codec.http2`
 * `io.netty.codec.memcache`
 * `io.netty.codec.mqtt`
 * `io.netty.codec.redis`
 * `io.netty.codec.smtp`
 * `io.netty.codec.socks`
 * `io.netty.codec.stomp`
 * `io.netty.codec.xml`
 * `io.netty.common`
 * `io.netty.handler`
 * `io.netty.handler.proxy`
 * `io.netty.resolver`
 * `io.netty.resolver.dns`
 * `io.netty.transport`
 * `io.netty.transport.epoll` (`native` omitted - reserved keyword in Java)
 * `io.netty.transport.kqueue` (`native` omitted - reserved keyword in Java)
 * `io.netty.transport.unix.common` (`native` omitted - reserved keyword in Java)
 * `io.netty.transport.rxtx`
 * `io.netty.transport.sctp`
 * `io.netty.transport.udt`



Automatic modules do not provide any means to declare dependencies, so you need to list each used module separately
in your `module-info` file.
