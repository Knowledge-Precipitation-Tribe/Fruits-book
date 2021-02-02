# 什么是IO
IO在计算机中指的是Input/Output即输入和输出，I/O分为两个阶段：
1. 数据准备阶段(Waiting for the data to be ready)  
2. 内核空间复制数据到用户进程缓冲（用户空间）阶段 (Copying the data from the kernel to the process)

拿网络IO来说，等待的过程就是数据从网络到网卡再到内核空间。读写的过程就是内核空间和用户空间的相互拷贝。

大多数文件系统的默认 I/O 操作都是缓存 I/O。在Linux的缓存I/O机制中，操作系统会将I/O的数据缓存在文件系统的页缓存(page cache)中，也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间也就是用户空间 [[用户态与内核态]]。

但是数据在传输过程中需要在用户空间和内核空间进行多次数据拷贝操作，这些数据拷贝操作所带来的CPU以及内存开销是非常大的。

# IO模式
前面我们介绍过I/O分为两个阶段，正式因为这两个阶段，linux系统产生了下面五种网络模式的方案。  
- 阻塞I/O（blocking IO）  
- 非阻塞I/O（nonblocking IO）  
- [[IO多路复用]]（IO multiplexing）  
- 信号驱动I/O（signal driven IO）  
- 异步I/O（asynchronous IO）

注：由于signal driven IO在实际中并不常用，所以我这只提及剩下的四种IO Model。

# 阻塞IO
在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样：

当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据（对于网络IO来说，很多时候数据在一开始还没有到达。比如，还没有收到一个完整的UDP包。这个时候kernel就要等待足够的数据到来）。这个过程需要等待，也就是说数据被拷贝到操作系统内核的缓冲区中是需要一个过程的。而在用户进程这边，整个进程会被阻塞（当然，是进程自己选择的阻塞）。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。
![](img/blockingio.png)
> 所以，blocking IO的特点就是在IO执行的两个阶段都被block了。

## 同步IO
同步IO是指发起IO请求后，必须拿到IO的数据才可以继续执行。按照程序的表现分为两种：
- 在等待数据的过程中，和拷贝数据的过程中，线程都在阻塞，这就是同步阻塞IO。
- 在等待数据的过程中，线程采用死循环式轮询，在拷贝数据的过程中，线程在阻塞，这其实还是同步阻塞IO。

在IO上，同步和非阻塞是互斥的，所以不存在同步非阻塞IO。

## 异步阻塞IO
异步IO是指发起IO请求后，不用拿到IO的数据就可以继续执行。用户线程的继续执行，和操作系统准备IO数据的过程是同时进行的，因此才叫做异步IO。

在等待数据的过程中，用户线程继续执行，在拷贝数据的过程中，线程在阻塞，这就是异步阻塞IO。在这种情况下用户线程没有参与数据等待的过程，所以它是异步的，但用户线程参与了数据拷贝的过程，所以它又是阻塞的。合起来就是异步阻塞IO。

# 非阻塞IO
linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：
![](https://tva1.sinaimg.cn/large/008eGmZEly1gn9cdtcy29j30gr0993z8.jpg)

当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

> 所以，nonblocking IO的特点是用户进程需要**不断的主动询问**kernel数据好了没有。

## 异步非阻塞IO
在等待数据的过程中，和拷贝数据的过程中，用户线程都在继续执行，这就是异步非阻塞IO。在这种情况下用户线程既没有参与等待过程也没有参与拷贝过程，所以它是异步的，当它接到通知时，数据已经准备好了，它没有因为IO数据而阻塞过，所以它又是非阻塞的，合起来就是异步非阻塞IO。

# IO多路复用
IO multiplexing就是我们说的select，poll，epoll，有些地方也称这种IO方式为event driven IO。select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select，poll，epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。
![](https://tva1.sinaimg.cn/large/008eGmZEly1gn9chut38fj30gx092aao.jpg)

当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

> 所以，I/O 多路复用的特点是通过一种机制一个进程能同时等待多个[[文件描述符]]，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，select()函数就可以返回。

这个图和blocking IO的图其实并没有太大的不同，事实上，还更差一些。因为这里需要使用两个system call (select 和 recvfrom)，而blocking IO只调用了一个system call (recvfrom)。但是用select的优势在于它可以同时处理多个connection。

所以，如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。）

在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block。

# 异步IO
inux下的asynchronous IO其实用得很少。先看一下它的流程：
![](https://tva1.sinaimg.cn/large/008eGmZEly1gn9cjpdhtij30fw090wey.jpg)
用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

**各个IO Model的比较如图所示：**  
![](https://tva1.sinaimg.cn/large/008eGmZEly1gn9cqxvmfwj30h2093q3s.jpg)

通过上面的图片，可以发现non-blocking IO和asynchronous IO的区别还是很明显的。在non-blocking IO中，虽然进程大部分时间都不会被block，但是它仍然要求进程去主动的check，并且当数据准备完成以后，也需要进程主动的再次调用recvfrom来将数据拷贝到用户内存。而asynchronous IO则完全不同。它就像是用户进程将整个IO操作交给了他人（kernel）完成，然后他人做完后发信号通知。在此期间，用户进程不需要去检查IO操作的状态，也不需要主动的去拷贝数据。