- select 基于轮训机制,单个进程可监视的fd数量被限制

  它仅仅知道了，有I/O事件发生了，却并不知道是哪那几个流（可能有一个，多个，甚至全部），我们只能无差别轮询所有流，找出能读出数据，或者写入数据的流，对他们进行操作。所以**select具有O(n)的无差别轮询复杂度**，同时处理的流越多，无差别轮询时间就越长。

  内核需要将消息传递到用户空间，都需要内核拷贝动作

  

- poll

  poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态， **但是它没有最大连接数的限制**，原因是它是基于链表来存储的.

  

- epoll基于操作系统支持的I/O通知机制 epoll支持水平触发和边沿触发两种模式。

  epoll通过内核和用户空间共享一块内存来实现的。

