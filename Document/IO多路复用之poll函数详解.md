
[poll](https://blog.csdn.net/lixungogogo/article/details/52226434)

poll的机制与select类似，与select在本质上没有多大差别，管理多个描述符也是进行轮询，根据描述符的状态进行处理，
		
		但是poll没有最大文件描述符数量的限制。 
　　poll和select同样存在一个缺点就是，包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大。

poll函数
　　函数格式如下所示：

		#include <poll.h>
####int poll ( struct pollfd * fds, unsigned int nfds, int timeout);

参数分析
struct pollfd * fds
pollfd结构体定义如下：

	struct pollfd {
	    int fd;               /* 文件描述符 */
	    short events;         /* 等待的事件 */
	    short revents;        /* 实际发生了的事件 */
	} ; 

　　每一个pollfd结构体指定了一个被监视的文件描述符，可以传递多个结构体，指示poll()监视多个文件描述符。 
　　每个结构体的events域是监视该文件描述符的事件掩码，由用户来设置这个域。 
　　revents域是文件描述符的操作结果事件掩码，内核在调用返回时设置这个域。 
　　events域中请求的任何事件都可能在revents域中返回。 
　　合法的事件如下：

	   POLLIN 　　　　　　　　有数据可读。
	　　POLLRDNORM 　　　　  有普通数据可读。
	　　POLLRDBAND　　　　　 有优先数据可读。
	　　POLLPRI　　　　　　　　 有紧迫数据可读。
	　　POLLOUT　　　　　　      写数据不会导致阻塞。
	　　POLLWRNORM　　　　　  写普通数据不会导致阻塞。
	　　POLLWRBAND　　　　　   写优先数据不会导致阻塞。
	　　POLLMSGSIGPOLL 　　　　消息可用。

　　此外，revents域中还可能返回下列事件： 

	   POLLER　　   指定的文件描述符发生错误。
	　　POLLHUP　　 指定的文件描述符挂起事件。
	　　POLLNVAL　　指定的文件描述符非法。

　　这些事件在events域中无意义，因为它们在合适的时候总是会从revents中返回。

　　使用poll()和select()不一样，你不需要显式地请求异常情况报告。 
　　POLLIN | POLLPRI等价于select()的读事件. 
　　POLLOUT |POLLWRBAND等价于select()的写事件。 
　　POLLIN等价于POLLRDNORM |POLLRDBAND 
　　而POLLOUT则等价于POLLWRNORM。

　　例如，要同时监视一个文件描述符是否可读和可写，我们可以设置 events为POLLIN |POLLOUT。在poll返回时，我们可以检查revents中的标志，对应于文件描述符请求的events结构体。如果POLLIN事件被设置，则文件描述符可以被读取而不阻塞。如果POLLOUT被设置，则文件描述符可以写入而不导致阻塞。这些标志并不是互斥的：它们可能被同时设置，表示这个文件描述符的读取和写入操作都会正常返回而不阻塞。

unsigned int nfds,
　　nfds_t类型的参数，用于标记数组fds中的结构体元素的总数量

	int timeout
	　　timeout参数指定等待的毫秒数，无论I/O是否准备好，poll都会返回。 
	　　timeout指定为负数值表示无限超时，使poll()一直挂起直到一个指定事件发生 
	　　timeout为0   立即返回 并不等待 ， 指示poll调用立即返回并列出准备好I/O的文件描述符，但并不等待其它的事件。这种情况下，poll()就像它的名字那样，一旦选举出来，立即返回。

返回值
　　成功时，poll()返回结构体中revents域不为0的文件描述符个数。 
　　如果在超时前没有任何事件发生，poll()返回0。 
　　失败时，poll()返回-1，并设置errno为下列值之一： 
　　

	　　EBADF　　       一个或多个结构体中指定的文件描述符无效。
	　　EFAULTfds　　 指针指向的地址超出进程的地址空间。
	　　EINTR　　　　  请求的事件之前产生一个信号，调用可以重新发起。
	　　EINVALnfds　　参数超出PLIMIT_NOFILE值。
	　　ENOMEM　　     可用内存不足，无法完成请求。



#总结
　　poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，
如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。 
　　
   这个过程经历了多次无谓的遍历。 

	poll还有一个特点是“水平触发”，如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。 
　　

poll与select的不同，通过一个pollfd数组向内核传递需要关注的事件，故没有描述符个数的限制，
pollfd中的events字段和revents分别用于标示关注的事件和发生的事件，

	故pollfd数组只需要被初始化一次
 
　　poll的实现机制与select类似，其对应内核中的sys_poll，只不过poll向内核传递pollfd数组，然后对pollfd中的每个描述符进行poll，相比处理fdset来说，poll效率更高。poll返回后，需要对pollfd中的每个元素检查其revents值，来得指事件是否发生。

优点

	1）poll() 不要求开发者计算最大文件描述符加一的大小。 
	2）poll() 在应付大数目的文件描述符的时候速度更快，相比于select。 
	3）它没有最大连接数的限制，原因是它是基于链表来存储的。

缺点

	1）大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。 
	2）与select一样，poll返回后，需要轮询pollfd来获取就绪的描述符

基于poll的服务器代码

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
	
	int main(int argc,char* argv[])
	{
	    struct pollfd pfd[1];
	    int len = 1;
	
	    pfd[0].fd = 0;
	    pfd[0].events = POLLIN;
	    pfd[0].revents = 0;
	
	    int done = 0;
	    while(!done)
	    {
	        switch(poll(pfd,1,-1))
	        {
	            case 0:
	                printf("timeout\n");
	                break;
	            case -1:
	                perror("poll");
	                break;
	            default:
	            {
	                char buf[1024];
	                if(pfd[0].revents & POLLIN)
	                {
	                    ssize_t _s = read(pfd[0].fd,buf\
	                    sizeof(buf)-1);
	                    if(_s > 0)
	                    {
	                        buf[_s] = '\0';
	                        printf("echo:%s\n",buf);
	                    }
	                }
	            }
	            break;
	        }
	    }
	}
