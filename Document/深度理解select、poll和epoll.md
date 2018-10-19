
https://blog.csdn.net/mmbbz/article/details/70303951
https://blog.csdn.net/davidsguo008/article/details/73556811

# IO 复用

进程需要一种预先告知内核的能力， 使得内核一旦发现进程指定的一个或者多个IO就绪， 内核就通知进程，这种能力成为IO 复用

# IO复用的典型场景

1 多协议，既要支持tpc，又支持udp
2 既要处理监听套接字，又要处理已经连接的套接字

2

#IO 模型

1阻塞
2 非阻塞
3 io复用
4信号驱动

5 异步io  内核通知我们何时io，操作完成




#IO复用

是将阻塞，放置在select 或者poll上，而不是真正阻塞在io上，
这样 当io就绪后，再通过此调用，分辨出是哪些io就绪，进而进行处理


#select 
1 准备好读，
2 准备好写
3 异常
4 定时

以上情况发起通知

int  select ( int maxfdp1, 
				fd_set * readset ,fd_set * writeset ,fd_set * exceptset,  const struce timeval * timehou )
				
				
异常情况，支持两种
1 带外数据到达

fd_set通常为1024 ，

描述符集 ，整数数组，每个整数的每一位表示一个套接字
maxfdp1 带测试的描述符个数，是待测试的最大描述符+1




不敢兴趣的可以置空，全部置空，可以成为一个 sleep函数，比sleep更精确

###准备好读的场景
### 准备好写的场景

###套接字发生错误，将既可以读，又可以写

FD_SETSIZE 修改后，需要重新编译内核，所以改这个宏定义，通常没有作用



#带外数据

https://blog.csdn.net/ctthuangcheng/article/details/9498569
这是TCP紧急模式的一个重要特点：TCP首部指出发送端已经进入紧急模式（即伴随紧急偏移的URG标志已经设置），但是由紧急指针所指的实际数据字节却不一定随同送出。事实上即使发送端TCP因流量控制而暂停发送数据（接收端的套接字接收缓冲区已满，导致其TCP想发送端TCP通告了一个值为0 的窗口），紧急通知照样不伴随任何数据的发送。这也是应用进程使用TCP紧急模式（即带外数据）的一个原因：即便数据的流动会因为TCP的流量控制而停止，紧急通知却总是无障碍的发送到对端TCP。



#poll


int poll (struct pollfd * fdarray , unsigned long * , int timeout )  

pollfd
{
int  fd;
shore events;
short revents;


}


#IO多路复用之poll函数详解 

https://blog.csdn.net/lixungogogo/article/details/52226479

#IO多路复用之epoll函数详解 
https://blog.csdn.net/lixungogogo/article/details/52226479

工作模式
　　epoll对文件描述符的操作有两种模式：LT（level trigger）和ET（edge trigger）。LT模式是默认模式，LT模式与ET模式的区别如下：
LT模式：
　　当epoll_ wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。

ET模式：

　　当epoll_ wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。

　　ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。 
　　epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。 
　　那么ET模式是怎么做到的呢？

ET模式的原理

　　当一个socket句柄上有事件时，内核会把该句柄插入上面所说的准备就绪list链表，这时我们调用epoll_ wait，会把准备就绪的socket拷贝到用户态内存，然后清空准备就绪list链表。 
　　最后，epoll_ wait检查这些socket，如果不是ET模式（就是LT模式的句柄了），并且这些socket上确实有未处理的事件时，又把该句柄放回到刚刚清空的准备就绪链表了。 
　　所以，非ET的句柄，只要它上面还有事件，epoll_ wait每次都会返回。而ET模式的句柄，除非有新中断到，即使socket上的事件没有处理完，也是不会次次从epoll_wait返回的。

---------------------
作者：长着胡萝卜须的栗子 
来源：CSDN 
原文：https://blog.csdn.net/lixungogogo/article/details/52226479?utm_source=copy 
版权声明：本文为博主原创文章，转载请附上博文链接！

1）支持一个进程打开大数目的socket描述符(FD)

　　select最不能忍受的是一个进程所打开的FD是有一定限制的，由FD_SETSIZE设置，默认值是1024/2048。对于那些需要支持的上万连接数目的IM服务器来说显然太少了。这时候你一是可以选择修改这个宏然后重新编译内核。不过 epoll则没有这个限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左右，具体数目可以cat /proc/sys/fs/file-max察看,一般来说这个数目和系统内存关系很大。

2）IO效率不随FD数目增加而线性下降

　　传统的select/poll另一个致命弱点就是当你拥有一个很大的socket集合，不过由于网络延时，任一时间只有部分的socket是”活跃”的，但是select/poll每次调用都会线性扫描全部的集合，导致效率呈现线性下降。但是epoll不存在这个问题，它只会对”活跃”的socket进行操作—这是因为在内核实现中epoll是根据每个fd上面的callback函数实现的。那么，只有”活跃”的socket才会主动的去调用 callback函数，其他idle状态socket则不会，在这点上，epoll实现了一个”伪”AIO，因为这时候推动力在Linux内核。 
3）使用mmap加速内核与用户空间的消息传递。

　　这点实际上涉及到epoll的具体实现了。无论是select,poll还是epoll都需要内核把FD消息通知给用户空间，如何避免不必要的内存拷贝就很重要，在这点上，epoll是通过内核与用户空间mmap同一块内存实现的。



#select()和poll() IO多路复用模型

select的缺点：单个进程能够监视的文件描述符的数量存在最大限制，通常是1024，当然可以更改数量，但由于select采用轮询的方式扫描文件描述符，文件描述符数量越多，性能越差；(在linux内核头文件中，有这样的定义：#define __FD_SETSIZE??? 1024)内核 / 用户空间内存拷贝问题，select需要复制大量的句柄数据结构，产生巨大的开销；select返回的是含有整个句柄的数组，应用程序需要遍历整个数组才能发现哪些句柄发生了事件；select的触发方式是水平触发，应用程序如果没有完成对一个已经就绪的文件描述符进行IO操作，那么之后每次select调用还是会将这些文件描述符通知进程。


相比select模型，poll使用链表保存文件描述符，因此没有了监视文件数量的限制，但其他三个缺点依然存在。
拿select模型为例，假设我们的服务器需要支持100万的并发连接，则在__FD_SETSIZE 为1024的情况下，则我们至少需要开辟1k个进程才能实现100万的并发连接。除了进程间上下文切换的时间消耗外，从内核/用户空间大量的无脑内存拷贝、数组轮询等，是系统难以承受的。因此，基于select模型的服务器程序，要达到10万级别的并发访问，是一个很难完成的任务。


在之前我已经分析了这三个函数，请看我之前的文章： http://blog.csdn.net/lixungogogo/article/details/52226501IO多路复用之select函数详解
IO多路复用之poll函数详解
IO多路复用之epoll函数详解

　　这篇文章只总结优缺点，以便面试时回答。
select优点
1）select()的可移植性更好，在某些Unix系统上不支持poll() 
2）select() 对于超时值提供了更好的精度：微秒，而poll是毫秒。
select缺点
1） 单个进程可监视的fd数量被限制。 
2） 需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。

3） 对fd进行扫描时是线性扫描。fd剧增后，IO效率较低，因为每次调用都对fd进行线性扫描遍历，所以随着fd的增加会造成遍历速度慢的性能问题 
4）select() 函数的超时参数在返回时也是未定义的，考虑到可移植性，每次在超时之后在下一次进入到select之前都需要重新设置超时参数。
poll
　　poll与select不同，通过一个pollfd数组向内核传递需要关注的事件，故没有描述符个数的限制，　 
　　pollfd中的events字段和revents分别用于标示关注的事件和发生的事件，故pollfd数组只需要被初始化一次。 
　　poll的实现机制与select类似，其对应内核中的sys_poll，只不过poll向内核传递pollfd数组，然后对pollfd中的每个描述符进行poll，相比处理fdset来说，poll效率更高。　

　　poll返回后，需要对pollfd中的每个元素检查其revents值，来得指事件是否发生。
poll优点
1）poll() 不要求开发者计算最大文件描述符加一的大小。 
2）poll() 在应付大数目的文件描述符的时候相比于select速度更快 
3）它没有最大连接数的限制，原因是它是基于链表来存储的。
poll缺点
1）大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。 
2）与select一样，poll返回后，需要轮询pollfd来获取就绪的描述符
epoll
　　epoll是Linux下多路复用IO接口select/poll的增强版本。

　　它能显著减少程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率，因为它不会复用文件描述符集合来传递结果而迫使开发者每次等待事件之前都必须重新准备要被侦听的文件描述符集合。

　　另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。 
　　epoll除了提供select/poll 那种IO事件的电平触发（Level Triggered）外，还提供了边沿触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少epoll_wait/epoll_pwait的调用，提高应用程序效率。

底层实现
  epoll在底层实现了自己的高速缓存区，并且建立了一个红黑树用于存放socket，另外维护了一个链表用来存放准备就绪的事件。
工作过程：
　　执行epoll_ create时，创建了红黑树和就绪链表，执行epoll_ ctl时，如果增加socket句柄，则检查在红黑树中是否存在，存在立即返回，不存在则添加到树干上，然后向内核注册回调函数，用于当中断事件来临时向准备就绪链表中插入数据。执行epoll_wait时立刻返回准备就绪链表里的数据即可。
epoll优点
1）支持一个进程打开大数目的socket描述符(FD)
2）IO效率不随FD数目增加而线性下降
3）使用mmap加速内核与用户空间的消息传递。
总结
（1）select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。 
　　而epoll其实也需要调用 epoll_ wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在 epoll_wait中进入睡眠的进程。　

　　虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的 时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间，这就是回调机制带来的性能提升。
（2）select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次，而epoll只要一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内 部定义的等待队列），这也能节省不少的开销

---------------------

本文来自 mmbbz 的CSDN 博客 ，全文地址请点击：https://blog.csdn.net/mmbbz/article/details/70303951?utm_source=copy 


