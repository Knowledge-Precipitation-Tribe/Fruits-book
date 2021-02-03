> 转载自：https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-netpoller/

Go 语言在网络轮询器中使用I/O多路复用模型处理I/O操作，但是他没有选择最常见的系统调用 select。虽然select也可以提供 I/O 多路复用的能力，但是使用它有比较多的限制：

-   监听能力有限 — 最多只能监听1024个文件描述符；
-   内存拷贝开销大 — 需要维护一个较大的数据结构存储文件描述符，该结构需要拷贝到内核中；
-   时间复杂度𝑂(𝑛) — 返回准备就绪的事件个数后，需要遍历所有的文件描述符；

为了提高IO多路复用的性能，不同的操作系统也都实现了自己的IO多路复用函数，例如：epoll，kqueue和evport等。Go 语言为了提高在不同操作系统上的IO操作性能，使用平台特定的函数实现了多个版本的网络轮询模块：

-   [`src/runtime/netpoll_epoll.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_epoll.go)
-   [`src/runtime/netpoll_kqueue.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_kqueue.go)
-   [`src/runtime/netpoll_solaris.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_solaris.go)
-   [`src/runtime/netpoll_windows.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_windows.go)
-   [`src/runtime/netpoll_aix.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_aix.go)
-   [`src/runtime/netpoll_fake.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_fake.go)

这些模块在不同平台上实现了相同的功能，构成了一个常见的树形结构。编译器在编译Go语言程序时，会根据目标平台选择树中特定的分支进行编译。如果目标平台是 Linux，那么就会根据文件中的 `// +build linux` 编译指令选择 [`src/runtime/netpoll_epoll.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_epoll.go) 并使用 `epoll` 函数处理用户的I/O操作。



# 来源
- [draveness](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-netpoller/)