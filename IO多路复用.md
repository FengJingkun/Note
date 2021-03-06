##  I/O模型

一个输入操作通常包括两个阶段：

- 等待数据准备好
- 将数据从内核空间复制到进程的用户空间

对于一个socket上的输入操作，第一步通常涉及等待数据从网络中到达。当等待数据到达时，网卡接收数据并向CPU发送一个硬件中断，通过中断处理将数据复制到内核缓冲区。第二步就是将数据从内核缓冲区复制到应用进程的缓冲区。

<br>

### 阻塞式IO

应用进程被阻塞，直至数据到达，并从内核空间复制到用户空间才返回。

recvfrom用于接收socket传来的数据，并复制到应用进程缓冲区。进程调用recvfrom后被阻塞，直至网络数据到达并复制到进程的用户空间后才返回。

<br>

### 非阻塞式IO

应用进程调用recvfrom，内核返回错误码，应用进程继续执行，但需要不断执行recvfrom来获知IO是否完成，即**轮询**。

由于OS需要处理更多的系统调用，因此这种模型的CPU利用率比较低。

<br>

### 信号驱动IO

应用进程使用sigaction系统调用，内核立即返回，应用进程继续执行，即在等待数据到达阶段应用进程是非阻塞的。数据到达后，内核向应用进程发送SIGIO信号，应用进程收到信号后调用recvfrom将数据从内核复制到用户空间。

相比于轮询式的非阻塞IO，信号驱动IO的CPU利用率更高。

<br>

### 异步IO

应用进程使用aio_read系统调用，内核立即返回，用户进程继续执行。当网络数据到达，内核将其复制到用户空间后在想应用进程发送信号。

与信号驱动IO的区别在于，后者在数据到达，可以开始IO时通知用户进程；异步IO则是在数据到达，并且复制到用户空间后才通知进程。

<br>

- **同步IO：数据从内核空间复制到用户空间时，应用进程会阻塞**。
- **异步IO：第二阶段不会阻塞**。

**同步IO包括阻塞式IO，非阻塞式IO，信号驱动IO，它们的区别在第一个阶段；其中，非阻塞式IO和信号驱动IO的第一阶段不会阻塞。**

<br>

<br>

## IO复用

如果服务器没有IO复用，那么每一个socket连接都需要创建一个线程单独处理。如果同时有几万个连接，那么就需要创建相同数量的线程，这无疑是一笔非常大的开销。

**IO复用就是使用单个进程去管理多个IO事件，又称为事件驱动IO。**

select，poll和epoll都是IO多路复用的具体实现。