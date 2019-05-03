# linux下write操作原理、错误原因及处理方法

[原文链接](https://blog.csdn.net/king16304/article/details/52355051)

## 1. write函数介绍

### 文件I/O与标准I/O之争

根据《UNIX环境高级编程》中介绍，文件I/O与标准I/O之间的差别主要有以下几点：其一，文件I/O是在系统的内核中实现的，而标准I/O函数则提供了文件I/O函数的一个完整的带缓冲的替代品。因此说，文件I/O是一种较低级的I/O操作函数，而标准I/O则是一种相对较高的I/O。标准I/O函数将打开的文件模型抽象成“文件流”。常见的文件I/O有：open, write, create, close, read等；标准I/O有：fopen,freopen, fdopen, getc等。第二，标准I/O一般是磁盘和终端设备I/O的首选，而当试图用于网络I/O，即对网络socket进行操作时，应该应用文件I/O。

### 文件I/O的阻塞与非阻塞（以write函数为例）

前面已经说过，标准I/O是一种带有缓冲区的操作函数，而在实际的操作过程中，势必会出现阻塞与非阻塞相关的问题。下面以write操作来介绍阻塞与非阻塞这两种情况：

![](http://up.2cto.com/2012/0810/20120810021015418.jpg)

write函数首先将进程需要发送的数据先放在进程缓冲区中，然后向socket的发送缓冲区进行拷贝，在此，可能出现这样情况，即当进程缓冲区中的数据量大于此时发送缓冲区中所能接受的数据量时，若此时处于阻塞模式，应用进程将会被挂起，直到进程缓冲区中的数据全部拷贝到发送缓冲区中，注意此时内核也不会返回write函数，因此，在阻塞模式下，若write函数正常返回，这也并不代表数据已经完成被对方进程接收，至多只能说明数据已经被发送缓冲区完全接受；若是处于非阻塞模式，此时write操作将会失败，内核会立即返回EAGAIN错误，在此需要声明的是，有时候在某些地方说会返回EWOULDBLOCK错误，其实二者本质一样，只是分别用于不同的系统罢了，前者主要是出现于GNU系统，，而后者主要出现在类BSD系统。

【引申1】阻塞与非阻塞的转换：切换socket fd的阻塞标志。

	int  fcntl(int fd , int cmd)
	int  fcntl(int fd,int cmd,long arg)

其中cmd代表要操作的命令，常见有：

- F_GETFL：取得fd的当前状态标志
- F_SETFL：设置fd的当前状态标志

	[cpp]
	flags = (long) fcntl(pc->fd, F_GETFL);  
	bflags = flags & ~O_NONBLOCK; /* clear non-block flag, i.e. block */  
	fcntl(pc->fd, F_SETFL, bflags);  

【引申2】Linux中发送缓存大小的查看：

	sysctl -a | grep net.ipv4.tcp_wmem
	net.ipv4.tcp_wmem = 4096 16384 81920 （这三个值分别代表发送缓冲区的最少字节数，默认字节数以及最多字节数）

## 2. write常见错误以及原因分析

前面已经说过EAGAIN错误出现的原因，下面主要讲解EPIPE错误是在何种情景下产生的。

我们知道TCP连接需要三次握手，而退出需要四个过程，而EPIPE则是产生于进程socket的退出过程中，对应上面的原理图，若B端的进程已经主动关闭（发送FIN），但是A端因为各种原因（主要是未同步），未能知晓并仍然向对方发送数据，此时A端内核会返回EPIPE错误，它会发送SIGPIPE信号给进程A，默认情况下，进程将会自动退出。
    
最近在做项目过程中，因为Apache server端的keep-alive配置时间过短，导致过早发出FIN，而使得client端的socket出现EPIPE错误，最后将keep-alive时间配置稍长点，一切问题OK！