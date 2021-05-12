# BIO、NIO和AIO

### Unix五种IO模型

* ##### 阻塞式I/O：blocking IO

* ##### 非阻塞式I/O：nonblocking IO

* ##### I/O复用（select，poll，epoll...）: IO multiplexing

* ##### 信号驱动式I/O（SIGIO）：signal driven IO

* ##### 异步I/O（POSIX的aio_系列函数）：asynchronous IO

对于这五种IO模型，Java并不是一开始就都全部支持的，Java的演化过程：

1. **JDK1.4之前**，Java的IO模型只支持阻塞式IO，简称BIO；
2. **JDK1.4时**，Java支持来I/O多路复用模型，对比之前的IO模型，这是一个新的模型，所以称之为NIO（None-Blocking IO），即非阻塞IO。
3. **JDK1.7时**，对NIO包进行了升级，支持了异步I/O（Asynchronous IO)，简称为AIO。

### IO模型的基本概念

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/IO模型.png)

如图所示，以一个网络IO来举例，对于一个network IO (以read举例)，它会涉及到两个系统对象：一个是调用这个IO的进程，另一个就是系统内核(kernel)。当一个read操作发生时，它会经历两个阶段：

**阶段1：**等待数据准备 (Waiting for the data to be ready)

**阶段2：** 将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)

#### 两个概念

**用户空间**是常规进程所在区域。 JVM 就是常规进程，驻守于用户空间。用户空间是非特权区域：比如，在该区域执行的代码就不能直接访问硬件设备。

**内核空间**是操作系统所在区域。内核代码有特别的权力：它能与设备控制器通讯，控制着用户区域进程的运行状态，等等。最重要的是，所有 I/O 都直接（如这里所述）或间接通过内核空间。

### 1、Blocking IO

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/BlockingIO.png)

过程解释：

第一步通常涉及等待数据从网络中到达。当所有等待数据到达时，它被复制到内核中的某个缓冲区。

第二步就是把数据从内核缓冲区复制到应用程序缓冲区。

小结：**blocking IO的特点就是在IO执行的两个阶段都被block了**

### 2、非阻塞式I/O

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/NIO.png)

过程解释：

1. 当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。 从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。
2. 用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次 发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

小结：**用户进程第一个阶段不是阻塞的,需要不断的主动询问kernel数据好了没有；第二个阶段依然总是阻塞的。**

### 3、I/O多路复用

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/I:O多路复用.png)

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/I:O多路复用结构图.png)

IO复用同非阻塞IO本质一样，不过利用了新的select系统调用，由内核来负责本来是请求进程该做的轮询操作。看似比非阻塞IO还多了一个系统调用开销，不过因为可以支持多路IO，才算提高了效率。

它的基本原理就是select /epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。

IO 多路复用模型中，线程首先发起 select 调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起 read 调用。read 调用的过程（数据从内核空间->用户空间）还是阻塞的。

> 目前支持 IO 多路复用的系统调用，有 select，epoll 等等。select 系统调用，是目前几乎在所有的操作系统上都有支持
>
> - **select 调用** ：内核提供的系统调用，它支持一次查询多个系统调用的可用状态。几乎所有的操作系统都支持。
> - **epoll 调用** ：linux 2.6 内核，属于 select 调用的增强版本，优化了 IO 的执行效率。

**IO 多路复用模型，通过减少无效的系统调用，减少了对 CPU 资源的消耗。**

### 4、信号驱动式I/O

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/信号驱动式I:O.png)

### 5、异步I/O

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/异步I:O.png)

这类函数的工作机制是告知内核启动某个操作，并让内核在整个操作（包括将数据从内核拷贝到用户空间）完成后通知我们。

用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都 完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。 在这整个过程中，进程完全没有被block。



BIO、NIO、AIO

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/BIO、NIO和AIO.png)

### 参考

[Unix五种IO模型](http://www.tianshouzhi.com/api/tutorials/netty/221)

[IO模型](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/basis/IO%E6%A8%A1%E5%9E%8B.md)

