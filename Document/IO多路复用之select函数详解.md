
https://blog.csdn.net/lixungogogo/article/details/52219951

IO复用
　　我们首先来看看服务器编程的模型，客户端发来的请求服务端会产生一个进程来对其进行服务，每当来一个客户请求就产生一个进程来服务，然而进程不可能无限制的产生，因此为了解决大量客户端访问的问题，引入了IO复用技术。 
　　即：一个进程可以同时对多个客户请求进行服务。

　　也就是说IO复用的“介质”是进程(准确的说复用的是select和poll，因为进程也是靠调用select和poll来实现的)，复用一个进程(select和poll)来对多个IO进行服务，虽然客户端发来的IO是并发的但是IO所需的读写数据多数情况下是没有准备好的，因此就可以利用一个函数(select和poll)来监听IO所需的这些数据的状态，一旦IO有数据可以进行读写了，进程就来对这样的IO进行服务。 
　　IO多路复用指内核一旦发现进程指定的一个或者多个IO条件准备读取，它就通知该进程。 
　　IO多路复用适用如下场合： 
　　1.当客户处理多个描述字时（一般是交互式输入和网络套接口），必须使用I/O复用。 
　　2.当一个客户同时处理多个套接口时，而这种情况是可能的，但很少出现。 
　　3.如果一个TCP服务器既要处理监听套接口，又要处理已连接套接口，一般也要用到I/O复用。 
　　4.如果一个服务器即要处理TCP，又要处理UDP，一般要使用I/O复用。 
　　5.如果一个服务器要处理多个服务或多个协议，一般要使用I/O复用。

　　与多进程和多线程技术相比，I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销。

select函数
　　该函数允许进程指示内核等待多个事件中的任何一个发生，并只在有一个或多个时间发生或经历一段指定的时间后才唤醒他。 
　　

#include <sys/select.h>
#include >sys/time.h>

int select(int maxfdp1,fd_set *readset,fd_set *writeset,fd_set *exceptset,const struct timeval *timeout);
1
2
3
4
返回值：
　　若有就绪描述符返回其数目，若超时则为0，若出错则为-1

参数
第一个参数——int maxfdp1
　　第一个参数maxfdp1指定待测试的描述字个数。 
　　它的值是待测试的最大描述字加1（因此把该参数命名为maxfdp1），描述字0、1、2…maxfdp1-1均将被测试。 
因为文件描述符是从0开始的。

fd_set *readset
fd_set *writeset
fd_set *exceptset
　　中间的三个参数readset、writeset和exceptset指定我们要让内核测试读、写和异常条件的描述字。　 
　　如果对某一个的条件不感兴趣，就可以把它设为空指针。struct fd_set可以理解为一个集合，这个集合中存放的是文件描述符，可通过以下四个宏进行设置：

void FD_ZERO(fd_set *fdset);           
//清空集合

void FD_SET(int fd, fd_set *fdset);   
//将一个给定的文件描述符加入集合之中

void FD_CLR(int fd, fd_set *fdset);   
//将一个给定的文件描述符从集合中删除

int FD_ISSET(int fd, fd_set *fdset);   
// 检查集合中指定的文件描述符是否可以读写 

	const struct timeval *timeout
	　　timeout告知内核等待所指定描述字中的任何一个就绪可花多少时间。其timeval结构用于指定这段时间的秒数和微秒数。
	
	     struct timeval{
	
	               long tv_sec;   //seconds
	
	               long tv_usec;  //microseconds
	
	   };
1
2
3
4
5
6
7
这个参数有三种可能： 
１．永远等待下去：仅在有一个描述字准备好I/O时才返回。为此，把该参数设置为空指针NULL。 
２．等待一段固定时间：在有一个描述字准备好I/O时返回，但是不超过由该参数所指向的timeval结构中指定的秒数和微秒数。 
3.根本不等待：检查描述字后立即返回，这称为轮询。为此，该参数必须指向一个timeval结构，而且其中的定时器值必须为0。

select函数的调用过程
（1）使用copy_from_user从用户空间拷贝fd_set到内核空间

（2）注册回调函数__pollwait

（3）遍历所有fd

　　调用其对应的poll方法（对于socket，这个poll方法是sock_poll，sock_poll根据情况会调用到tcp_poll,udp_poll或者datagram_poll）

（4）以tcp_poll为例，其核心实现就是__pollwait，也就是上面注册的回调函数。

（5）__pollwait的主要工作就是把current（当前进程）挂到设备的等待队列中，不同的设备有不同的等待队列，对于tcp_poll来说，其等待队列是sk->sk_sleep（注意把进程挂到等待队列中并不代表进程已经睡眠了）。在设备收到一条消息（网络设备）或填写完文件数据（磁盘设备）后，会唤醒设备等待队列上睡眠的进程，这时current便被唤醒了。

（6）poll方法返回时会返回一个描述读写操作是否就绪的mask掩码，根据这个mask掩码给fd_set赋值。

（7）如果遍历完所有的fd，还没有返回一个可读写的mask掩码，则会调用schedule_timeout是调用select的进程（也就是current）进入睡眠。当设备驱动发生自身资源可读写后，会唤醒其等待队列上睡眠的进程。如果超过一定的超时时间（schedule_timeout指定），还是没人唤醒，则调用select的进程会重新被唤醒获得CPU，进而重新遍历fd，判断有没有就绪的fd。

（8）把fd_set从内核空间拷贝到用户空间。

select睡眠和唤醒过程
　　select巧妙的利用等待队列机制让用户进程适当在没有资源可读/写时睡眠，有资源可读/写时唤醒。

select睡眠过程
　　select会循环遍历它所监测的fd_　set内的所有文件描述符对应的驱动程序的poll函数。 
　　驱动程序提供的poll函数首先会将调用select的用户进程插入到该设备驱动对应资源的等待队列(如读/写等待队列)，然后返回一个bitmask告诉select当前资源哪些可用。　 
　　当select循环遍历完所有fd_set内指定的文件描述符对应的poll函数后，如果没有一个资源可用(即没有一个文件可供操作)，则select让该进程睡眠，一直等到有资源可用为止，进程被唤醒(或者timeout)继续往下执行。

select唤醒过程
　　唤醒该进程的过程通常是在所监测文件的设备驱动内实现的。 
　　驱动程序维护了针对自身资源读写的等待队列。当设备驱动发现自身资源变为可读写并且有进程睡眠在该资源的等待队列上时，就会唤醒这个资源等待队列上的进程。

# select的缺点
　　　1.每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大 
　　2.同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大 
　　3.select支持的文件描述符数量太小了，默认是1024


使用select函数写的服务器代码如下：

	int fds[128];
	const int len = 128;
	
	
	int startup(char *ip,int port)
	{
	    int listen_sock = socket(AF_INET,SOCK_STREAM,0);
	    struct sockaddr_in local;
	    local.sin_family = AF_INET;
	    local.sin_addr.s_addr = inet_addr(ip);
	    local.sin_port = port;
	    bind(&listen_sock,(struct sockaddr*)&local,sizeof(local));
	    if(listen(listen_sock,5) < 0)
	    {
	        perror("listen");
	    }
	    return listen_sock;
	}
	
	int main()
	{
	    if(argc != 3)
	    {
	        usage(argv[0]);
	        exit(1);
	    }
	    int i = 0;
	    for(;i < len;i++)
	    {
	        fds[i] = -1;
	    }
	
	    int listen_sock = startup(argv[1],atoi(argv[2]));
	
	    fd_set rfds;
	    fds[0] = listen_sock;
	    int done;
	    while(!done)
	    {
	        int max_fd = -1;
	        FD_ZERO(&rfds);
	        for(int i = 0;i < len;i++)
	        {
	            if(fds[i] != -1)
	            {
	                FD_SET(fds[i],&rfds);
	            }
	            max_fd = fd[i] > max_fd ? fd[i]:max_fd;
	        }
	        struct timeval timeout = {5,0};
	        switch(select(max_fd+1,&rfds,NULL,NULL,NULL))
	        {
	            case 0:
	            printf("timeout\n");
	            break;
	            case -1:
	            perror("select");
	            break;
	            default:
	            {
	                if(FD_ISSET(listen_sock,&rfds))
	                {
	                    struct sockaddr_in peer;
	                    int len = sizeof(peer);
	                    int new_fd = accept(listen_sock,\
	                    (struct sockaddr*)&peer,&len);
	                    if(new_fd > 0)
	                    {
	                        printf("get a new client->%s:%d\n",\
	                        inet_addr(peer.sin_addr),\
	                        ntohs(peer.sin_port));
	                        for(int i = 0; i < len;i++)
	                        {
	                            if(fds[i] == -1)
	                            {
	                                fds[i] = new_fd;
	                                break;
	                            }
	                        }
	                        if(i == len)
	                        {
	                            close(new_fd);
	                        }
	                    }
	                }
	                else
	                {
	                    char buf[1024];
	                    for(int i = 0; i < len;i++)
	                    {
	                        if(i != 0 && FD_ISSET(fds[i],\
	                                        &rfds))
	                        {
	                            ssize_t _s = read(fds[i],buf,sizeof(buf))
	                            if(_s > 0)
	                            {
	                                buf[_s] = '\0';
	                                printf("client:%s\n",buf);
	                            }
	                            else if(_s == 0)
	                            {
	                                printf("client:%d is close\n",fds[i]);
	                                close(fds[i]);
	                                fds[i] = -1;
	                            }
	                            else 
	                                perror("read");
	                        }
	                    }
	                }
	            }
	            break;          
	        }
	    }
	}
