**<span style="font-size: 35px;">🥭 Java IO IO模型</span>**

---

## 概述

在前面[Java IO - 基础知识](java/IO/基础知识)一文中，我们知道了I/O（**I**nput/**O**utpu） 指的是**输入／输出** 。

### 从计算机结构角度

在这里我们进一步从**计算机结构**的角度来解读一下 I/O：

根据冯.诺依曼结构，计算机结构分为 5 大部分：`运算器`、`控制器`、`存储器`、`输入设备`、`输出设备`。

![计算机结构](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/27646844-9f74-46bf-86de-18c8173bce9e_计算机结构.jpg)

- 输入设备（比如键盘）和输出设备（比如显示器）都属于外部设备。网卡、硬盘这种既可以属于输入设备，也可以属于输出设备。
- 输入设备向计算机输入数据，输出设备接收计算机输出的数据。

从计算机结构的视角来看的话，**I/O 描述了计算机系统与外部设备之间通信的过程。**

### 从应用程序角度

我们再先从应用程序的角度来解读一下 I/O：

根据操作系统相关的知识：为了保证操作系统的稳定性和安全性，一个进程的地址空间划分为 **用户空间（User space）** 和 **内核空间（Kernel space ）** 。

像我们平常运行的应用程序都是运行在用户空间，只有内核空间才能进行系统态级别的资源有关的操作，比如文件管理、进程通信、内存管理等等。也就是说，**我们想要进行 IO 操作，一定是要依赖内核空间的能力**。

并且，用户空间的程序不能直接访问内核空间。

当想要执行 IO 操作时，由于没有执行这些操作的权限，只能发起系统调用请求操作系统帮忙完成。

因此，用户进程想要执行 IO 操作的话，必须通过 **系统调用** 来间接访问内核空间

我们在平常开发过程中接触最多的就是 **磁盘 IO（读写文件）** 和 **网络 IO（网络请求和响应）**。

**从应用程序的视角来看的话，我们的应用程序对操作系统的内核发起 IO 调用（系统调用），操作系统负责的内核执行具体的 IO 操作。也就是说，我们的应用程序实际上只是发起了 IO 操作的调用而已，具体 IO 的执行是由操作系统的内核来完成的。**

当应用程序发起 I/O 调用后，会经历两个步骤：

1. 内核等待 I/O 设备准备好数据
2. 内核将数据从内核空间拷贝到用户空间。

对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待分组到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用进程缓冲区。



## UNIX系统的IO模型

unix系统中，有5中IO模型，分别是：

- 同步阻塞I/O
- 同步非阻塞I/O
- I/O多路复用
- 信号驱动I/O（SIGIO）
- 异步I/O（AIO）



### 同步阻塞I/O

应用进程被阻塞，直到数据复制到应用进程缓冲区中才返回。

应该注意到，在阻塞的过程中，其它程序还可以执行，因此阻塞不意味着整个操作系统都被阻塞。因为其他程序还可以执行，因此不消耗 CPU 时间，这种模型的执行效率会比较高。

下图中，recvfrom 用于接收 Socket 传来的数据，并复制到应用进程的缓冲区 buf 中。这里把 recvfrom() 当成系统调用。

```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
```

![同步阻塞I/O](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/0d1bb486-9a06-4988-8915-4cabb15d9d48_java-io-model-0.png)



### 同步非阻塞I/O

应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询(polling)。

由于 CPU 要处理更多的系统调用，因此这种模型是比较低效的。

![同步非阻塞I/O](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/16bb4ef1-f96b-4ca8-8b3f-85ebd53f9d79_java-io-model-1.png)



### I/O多路复用

使用 select 或者 poll 等待数据，并且可以等待多个套接字中的任何一个变为可读，这一过程会被阻塞，当某一个套接字可读时返回。之后再使用 recvfrom 把数据从内核复制到进程中。

它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

如果一个 Web 服务器没有 I/O 复用，那么每一个 Socket 连接都需要创建一个线程去处理。如果同时有几万个连接，那么就需要创建相同数量的线程。并且相比于多进程和多线程技术，I/O 复用不需要进程线程创建和切换的开销，系统开销更小。

![I/O多路复用](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/e4f69459-10cd-4fa1-8cc3-b792b0ead3c0_java-io-model-2.png)



### 信号驱动 I/O

应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

![信号驱动 I/O](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/e98553d9-cfa3-4e6b-8845-c1cec0f4c20f_java-io-model-3.png)

### 异步 I/O

进行 aio_read 系统调用会立即返回，应用进程继续执行，不会被阻塞，内核会在所有操作完成之后向应用进程发送信号。

异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

![异步 I/O](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/4dfaa655-e6ca-4f31-8e57-ebabc0e560b1_java-io-model-4.png)



### I/O 模型比较

#### 同步 I/O 与异步 I/O

- 同步 I/O: 应用进程在调用 recvfrom 操作时会阻塞。
- 异步 I/O: 不会阻塞。

阻塞式 I/O、非阻塞式 I/O、I/O 复用和信号驱动 I/O 都是同步 I/O，虽然非阻塞式 I/O 和信号驱动 I/O 在等待数据阶段不会阻塞，但是在之后的将数据从内核复制到应用进程这个操作会阻塞。

#### 五大 I/O 模型比较

前四种 I/O 模型的主要区别在于第一个阶段，而第二个阶段是一样的: 将数据从内核复制到应用进程过程中，应用进程会被阻塞。

![五大 I/O 模型比较](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/0c4ae024-6768-401c-8c6b-fa6bb0fa57f1_五大IO模型比较.png)



### IO多路复用

> [!TIP]
>
> IO多路复用最为重要，后面的文章[Java IO - NIO模型详解](java/IO/NIO模型详解)将对IO多路复用，Ractor模型以及Java NIO对其的支持作详解。

这里主要概要性的理解: IO多路复用工作模式和应用。

#### IO多路复用工作模式

epoll 的描述符事件有两种触发模式: LT(level trigger)和 ET(edge trigger)。

##### 1. LT 模式

当 epoll_wait() 检测到描述符事件到达时，将此事件通知进程，进程可以不立即处理该事件，下次调用 epoll_wait() 会再次通知进程。是默认的一种模式，并且同时支持 Blocking 和 No-Blocking。

##### 2. ET 模式

和 LT 模式不同的是，通知之后进程必须立即处理事件，下次再调用 epoll_wait() 时不会再得到事件到达的通知。

很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 No-Blocking，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

#### 应用场景

很容易产生一种错觉认为只要用 epoll 就可以了，select 和 poll 都已经过时了，其实它们都有各自的使用场景。

##### 1. select 应用场景

select 的 timeout 参数精度为 1ns，而 poll 和 epoll 为 1ms，因此 select 更加适用于实时要求更高的场景，比如核反应堆的控制。

select 可移植性更好，几乎被所有主流平台所支持。

##### 2. poll 应用场景

poll 没有最大描述符数量的限制，如果平台支持并且对实时性要求不高，应该使用 poll 而不是 select。

需要同时监控小于 1000 个描述符，就没有必要使用 epoll，因为这个应用场景下并不能体现 epoll 的优势。

需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为 epoll 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且epoll 的描述符存储在内核，不容易调试。

##### 3. epoll 应用场景

只需要运行在 Linux 平台上，并且有非常大量的描述符需要同时轮询，而且这些连接最好是长连接。



## Java 常见 IO 模型

### BIO (Blocking I/O)

**BIO 属于同步阻塞 IO 模型** 。

同步阻塞 IO 模型中，应用程序发起 read 调用后，会一直阻塞，直到内核把数据拷贝到用户空间。

![同步阻塞 IO 模型](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/9b66c734-9415-48e3-8806-32da85607b46_同步阻塞.jpg)

在客户端连接数量不高的情况下，是没问题的。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。

> [!TIP]
>
> 想要继续深入了解BIO，请阅读[Java IO - BIO模型详解](java/IO/BIO模型详解)一文



### NIO (Non-blocking/New I/O)

Java 中的 NIO 于 Java 1.4 中引入，对应 `java.nio` 包，提供了 `Channel` , `Selector`，`Buffer` 等抽象。NIO 中的 N 可以理解为 Non-blocking，不单纯是 New。它是支持面向缓冲的，基于通道的 I/O 操作方法。 对于高负载、高并发的（网络）应用，应使用 NIO 。

Java 中的 NIO 可以看作是 **I/O 多路复用模型**。也有很多人认为，Java 中的 NIO 属于同步非阻塞 IO 模型。

跟着我的思路往下看看，相信你会得到答案！

我们先来看看 **同步非阻塞 IO 模型**。

![同步非阻塞 IO 模型](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/212b3004-f7a0-4365-85cc-97b37f5dde7a_同步非阻塞IO模型.jpg)

同步非阻塞 IO 模型中，应用程序会一直发起 read 调用，等待数据从内核空间拷贝到用户空间的这段时间里，线程依然是阻塞的，直到在内核把数据拷贝到用户空间。

相比于同步阻塞 IO 模型，同步非阻塞 IO 模型确实有了很大改进。通过轮询操作，避免了一直阻塞。

但是，这种 IO 模型同样存在问题：**应用程序不断进行 I/O 系统调用轮询数据是否已经准备好的过程是十分消耗 CPU 资源的。**

这个时候，**I/O 多路复用模型** 就上场了。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/25496c56-02fb-4cef-88fb-95bd31b66a97_IO多路复用模型.jpg)

IO 多路复用模型中，线程首先发起 select 调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起 read 调用。read 调用的过程（数据从内核空间 -> 用户空间）还是阻塞的。

> 目前支持 IO 多路复用的系统调用，有 select，epoll 等等。select 系统调用，目前几乎在所有的操作系统上都有支持。
>
> - **select 调用** ：内核提供的系统调用，它支持一次查询多个系统调用的可用状态。几乎所有的操作系统都支持。
> - **epoll 调用** ：linux 2.6 内核，属于 select 调用的增强版本，优化了 IO 的执行效率。

**IO 多路复用模型，通过减少无效的系统调用，减少了对 CPU 资源的消耗。**

Java 中的 NIO ，有一个非常重要的**选择器 ( Selector )** 的概念，也可以被称为 **多路复用器**。通过它，只需要一个线程便可以管理多个客户端连接。当客户端数据到了之后，才会为其服务。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/0a82efca-c555-47a6-8c69-303547732679_IO多路复用2.jpg)

> [!TIP]
>
> 想要继续深入了解NIO，请阅读[Java IO - NIO模型详解](java/IO/NIO模型详解)一文



### AIO (Asynchronous I/O)

AIO 也就是 NIO 2。Java 7 中引入了 NIO 的改进版 NIO 2,它是异步 IO 模型。

异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

![AIO](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/4c947a03-e4ec-48d6-87f8-033e348775dd_异步IO模型1.jpg)

目前来说 AIO 的应用还不是很广泛。Netty 之前也尝试使用过 AIO，不过又放弃了。这是因为，Netty 使用了 AIO 之后，在 Linux 系统上的性能并没有多少提升。

最后，来一张图，简单总结一下 Java 中的 BIO、NIO、AIO。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/fdeedf7f-aeee-4189-8555-58f9688f07d5_异步IO模型2.png)

> [!TIP]
>
> 想要继续深入了解AIO，请阅读[Java IO - AIO模型详解](java/IO/AIO模型详解)一文



## 参考文章

- [Java 全栈知识体系](https://www.pdai.tech/md/java/io/java-io-model.html)
- [JavaGuide](https://javaguide.cn/java/io/io-model.html)