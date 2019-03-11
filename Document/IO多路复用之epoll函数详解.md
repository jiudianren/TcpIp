epoll
　　 在linux的网络编程中，很长的时间都在使用select来做事件触发。
     在linux新的内核中，有了一种替换它的机制，就是epoll。 
　　
   相比于select，
   
    epoll最大的好处在于它不会随着监听fd数目的增长而降低效率。
   
  因为在内核中的select实现中，它是采用轮询来处理的，轮询的fd数目越多，自然耗时越多。 
相对于select和poll来说，epoll更加灵活，没有描述符限制。 


    epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。

epoll接口
   epoll操作过程需要三个接口，分别如下：

	#include <sys/epoll.h>
	int epoll_create(int size);
	int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
	int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);

　　
首先要调用epoll_create建立一个epoll对象。参数size是内核保证能够正确处理的最大句柄数，多于这个最大数时内核可不保证效果。 
　　epoll_ctl可以操作上面建立的epoll，例如，将刚建立的socket加入到epoll中让其监控，或者把 epoll正在监控的某个socket句柄移出epoll，不再监控它等等。 
　　epoll_wait在调用时，在给定的timeout时间内，当在监控的所有句柄中有事件发生时，就返回用户态的进程。

### int epoll_create(int size);
　　创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大。这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值。 
　　需要注意的是，当创建好epoll句柄后，它就是会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

### int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
　　epoll的事件注册函数，它不同与select()是在监听事件时告诉内核要监听什么类型的事件epoll的事件注册函数. 
　　它不同与select()是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。 
　　第一个参数是epoll_create()的返回值，第二个参数表示动作，用三个宏来表示：

	1 EPOLL_CTL_ADD：注册新的fd到epfd中；
	2 EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
	3 EPOLL_CTL_DEL：从epfd中删除一个fd；

　　第三个参数是需要监听的fd。 
　　第四个参数是告诉内核需要监听什么事，struct epoll_event结构如下：

	struct epoll_event {
	  __uint32_t events;  /* Epoll events */
	  epoll_data_t data;  /* User data variable */
	};

events可以是以下几个宏的集合：

	EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
	EPOLLOUT：表示对应的文件描述符可以写；
	EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
	EPOLLERR：表示对应的文件描述符发生错误；
	EPOLLHUP：表示对应的文件描述符被挂断；
	EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
	EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里

### int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
　　等待事件的产生，类似于select()调用。 
　　参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size。 
　　参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。 
　　epoll_wait范围之后应该是一个循环，遍历所有的事件。 
　　我们调用epoll_ wait时就相当于以往调用select/poll，但是这时却不用传递socket句柄给内核，因为内核已经在epoll_ctl中拿到了要监控的句柄列表。 
　　所以，实际上在你调用epoll_ create后，内核就已经在内核态开始准备帮你存储要监控的句柄了，每次调用epoll_ctl只是在往内核的数据结构里塞入新的socket句柄。 
　　在内核里，一切皆文件。所以，epoll向内核注册了一个文件系统，用于存储上述的被监控socket。当你调用epoll_create时，就会在这个虚拟的epoll文件系统里创建一个file结点。当然这个file不是普通文件，它只服务于epoll。

# epoll实现机制
　　epoll在被内核初始化时（操作系统启动），同时会开辟出epoll自己的内核高速cache区，用于安置每一个我们想监控的socket。 
　　这些socket会以红黑树的形式保存在内核cache里，以支持快速的查找、插入、删除。 
　　这个内核高速cache区，就是建立连续的物理内存页，然后在之上建立slab层。 
　　简单的说，就是物理上分配好你想要的size的内存对象，每次使用时都是使用空闲的已分配好的对象。 
　　epoll的高效就在于，当我们调用epoll_ctl往里塞入数十万(10k,100k（这个级别可以），1000k(这个级别达不到))个句柄时，epoll_ wait仍然可以飞快的返回，并有效的将发生事件的句柄给我们用户。 
　　这是由于我们在调用epoll_create时，内核除了帮我们在epoll文件系统里建了个file结点，在内核cache里建了个红黑树用于存储以后epoll_ctl传来的socket外，还会再建立一个list链表，用于存储准备就绪的事件. 
　　当epoll_ wait调用时，仅仅观察这个list链表里有没有数据即可。有数据就返回，没有数据就sleep，等到timeout时间到后即使链表没数据也返回。所以，epoll_wait非常高效。 
　　而且，通常情况下即使我们要监控百万计的句柄，大多一次也只返回很少量的准备就绪句柄而已，所以，epoll_wait仅需要从内核态copy少量的句柄到用户态而已。 
　　那么，这个准备就绪list链表是怎么维护的呢？ 
　　当我们执行epoll_ctl时，除了把socket放到epoll文件系统里file对象对应的红黑树上之外，还会给内核中断处理程序注册一个回调函数，告诉内核，如果这个句柄的中断到了，就把它放到准备就绪list链表里。 
　　所以，当一个socket上有数据到了，内核在把网卡上的数据copy到内核中后就来把socket插入到准备就绪链表里了。
　　如此，一颗红黑树，一张准备就绪句柄链表，少量的内核cache，就帮我们解决了大并发下的socket处理问题。 
　　执行epoll_ create时，创建了红黑树和就绪链表，执行epoll_ctl时，如果增加socket句柄，则检查在红黑树中是否存在，存在立即返回，不存在则添加到树干上，然后向内核注册回调函数，用于当中断事件来临时向准备就绪链表中插入数据。执行epoll_wait时立刻返回准备就绪链表里的数据即可。

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

优点
IO多路复用之select函数总结 
IO多路复用之poll函数总结 
　　前两篇文章中我们说过select和poll的缺点，现在我们谈谈epoll的优点：

1）支持一个进程打开大数目的socket描述符(FD)

　　select最不能忍受的是一个进程所打开的FD是有一定限制的，由FD_SETSIZE设置，默认值是1024/2048。对于那些需要支持的上万连接数目的IM服务器来说显然太少了。这时候你一是可以选择修改这个宏然后重新编译内核。不过 epoll则没有这个限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左右，具体数目可以cat /proc/sys/fs/file-max察看,一般来说这个数目和系统内存关系很大。

2）IO效率不随FD数目增加而线性下降

　　传统的select/poll另一个致命弱点就是当你拥有一个很大的socket集合，不过由于网络延时，任一时间只有部分的socket是”活跃”的，但是select/poll每次调用都会线性扫描全部的集合，导致效率呈现线性下降。但是epoll不存在这个问题，它只会对”活跃”的socket进行操作—这是因为在内核实现中epoll是根据每个fd上面的callback函数实现的。那么，只有”活跃”的socket才会主动的去调用 callback函数，其他idle状态socket则不会，在这点上，epoll实现了一个”伪”AIO，因为这时候推动力在Linux内核。 
3）使用mmap加速内核与用户空间的消息传递。

　　这点实际上涉及到epoll的具体实现了。无论是select,poll还是epoll都需要内核把FD消息通知给用户空间，如何避免不必要的内存拷贝就很重要，在这点上，epoll是通过内核与用户空间mmap同一块内存实现的。



https://www.cnblogs.com/ggjucheng/archive/2012/01/17/2324974.html

	
	#include  <unistd.h>
	#include  <sys/types.h>       /* basic system data types */
	#include  <sys/socket.h>      /* basic socket definitions */
	#include  <netinet/in.h>      /* sockaddr_in{} and other Internet defns */
	#include  <arpa/inet.h>       /* inet(3) functions */
	#include <sys/epoll.h> /* epoll function */
	#include <fcntl.h>     /* nonblocking */
	#include <sys/resource.h> /*setrlimit */
	
	#include <stdlib.h>
	#include <errno.h>
	#include <stdio.h>
	#include <string.h>
	
	
	
	#define MAXEPOLLSIZE 10000
	#define MAXLINE 10240
	int handle(int connfd);
	int setnonblocking(int sockfd)
	{
	    if (fcntl(sockfd, F_SETFL, fcntl(sockfd, F_GETFD, 0)|O_NONBLOCK) == -1) {
	        return -1;
	    }
	    return 0;
	}
	
	int main(int argc, char **argv)
	{
	    int  servPort = 6888;
	    int listenq = 1024;
	
	    int listenfd, connfd, kdpfd, nfds, n, nread, curfds,acceptCount = 0;
	    struct sockaddr_in servaddr, cliaddr;
	    socklen_t socklen = sizeof(struct sockaddr_in);
	    struct epoll_event ev;
	    struct epoll_event events[MAXEPOLLSIZE];
	    struct rlimit rt;
	    char buf[MAXLINE];
	
	    /* 设置每个进程允许打开的最大文件数 */
	    rt.rlim_max = rt.rlim_cur = MAXEPOLLSIZE;
	    if (setrlimit(RLIMIT_NOFILE, &rt) == -1) 
	    {
	        perror("setrlimit error");
	        return -1;
	    }
	
	
	    bzero(&servaddr, sizeof(servaddr));
	    servaddr.sin_family = AF_INET; 
	    servaddr.sin_addr.s_addr = htonl (INADDR_ANY);
	    servaddr.sin_port = htons (servPort);
	
	    listenfd = socket(AF_INET, SOCK_STREAM, 0); 
	    if (listenfd == -1) {
	        perror("can't create socket file");
	        return -1;
	    }
	
	    int opt = 1;
	    setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
	
	    if (setnonblocking(listenfd) < 0) {
	        perror("setnonblock error");
	    }
	
	    if (bind(listenfd, (struct sockaddr *) &servaddr, sizeof(struct sockaddr)) == -1) 
	    {
	        perror("bind error");
	        return -1;
	    } 
	    if (listen(listenfd, listenq) == -1) 
	    {
	        perror("listen error");
	        return -1;
	    }
	    /* 创建 epoll 句柄，把监听 socket 加入到 epoll 集合里 */
	    kdpfd = epoll_create(MAXEPOLLSIZE);
	    ev.events = EPOLLIN | EPOLLET;
	    ev.data.fd = listenfd;
	    if (epoll_ctl(kdpfd, EPOLL_CTL_ADD, listenfd, &ev) < 0) 
	    {
	        fprintf(stderr, "epoll set insertion error: fd=%d\n", listenfd);
	        return -1;
	    }
	    curfds = 1;
	
	    printf("epollserver startup,port %d, max connection is %d, backlog is %d\n", servPort, MAXEPOLLSIZE, listenq);
	
	    for (;;) {
	        /* 等待有事件发生 */
	        nfds = epoll_wait(kdpfd, events, curfds, -1);
	        if (nfds == -1)
	        {
	            perror("epoll_wait");
	            continue;
	        }
	        /* 处理所有事件 */
	        for (n = 0; n < nfds; ++n)
	        {
	            if (events[n].data.fd == listenfd) 
	            {
	                connfd = accept(listenfd, (struct sockaddr *)&cliaddr,&socklen);
	                if (connfd < 0) 
	                {
	                    perror("accept error");
	                    continue;
	                }
	
	                sprintf(buf, "accept form %s:%d\n", inet_ntoa(cliaddr.sin_addr), cliaddr.sin_port);
	                printf("%d:%s", ++acceptCount, buf);
	
	                if (curfds >= MAXEPOLLSIZE) {
	                    fprintf(stderr, "too many connection, more than %d\n", MAXEPOLLSIZE);
	                    close(connfd);
	                    continue;
	                } 
	                if (setnonblocking(connfd) < 0) {
	                    perror("setnonblocking error");
	                }
	                ev.events = EPOLLIN | EPOLLET;
	                ev.data.fd = connfd;
	                if (epoll_ctl(kdpfd, EPOLL_CTL_ADD, connfd, &ev) < 0)
	                {
	                    fprintf(stderr, "add socket '%d' to epoll failed: %s\n", connfd, strerror(errno));
	                    return -1;
	                }
	                curfds++;
	                continue;
	            } 
	            // 处理客户端请求
	            if (handle(events[n].data.fd) < 0) {
	                epoll_ctl(kdpfd, EPOLL_CTL_DEL, events[n].data.fd,&ev);
	                curfds--;
	
	
	            }
	        }
	    }
	    close(listenfd);
	    return 0;
	}
	int handle(int connfd) {
	    int nread;
	    char buf[MAXLINE];
	    nread = read(connfd, buf, MAXLINE);//读取客户端socket流
	
	    if (nread == 0) {
	        printf("client close the connection\n");
	        close(connfd);
	        return -1;
	    } 
	    if (nread < 0) {
	        perror("read error");
	        close(connfd);
	        return -1;
	    }    
	    write(connfd, buf, nread);//响应客户端  
	    return 0;
	}