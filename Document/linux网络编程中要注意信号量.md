linux网络编程中要注意信号量

https://blog.csdn.net/lp525110627/article/details/53351313


1、SIGPIPE

##产生原因：

TCP的"四次分手"关闭. TCP是全双工的信道,?可以看作两条单工信道, TCP连接两端的两个端点各负责一条.

当对端调用close时,本端只是收到FIN包.按照TCP协议的语义,表示对端只是关闭了其所负责的那一条单工信道。

也就是说,?因为TCP协议的限制,?一个端点无法获知对端的socket是调用了close还是shutdown。

对一个已经收到FIN包的socket调用read方法,如果接收缓冲已空,则返回0,这就是常说的表示连接关闭.

但第一次对其调用write方法时,如果发送缓冲没问题,会返回正确写入(发送).
但发送的报文会导致对端发送RST报文,因为对端的socket已经调用了close,完全关闭,既不发送,也不接收数据.所以,

第二次调用write方法(假设在收到RST之后),会生成SIGPIPE信号,导致进程退出。

##解决方法：

为了避免进程退出,?可以捕获SIGPIPE信号,?或者忽略它,?给它设置SIG_IGN信号处理函数

signal(SIGPIPE, SIG_IGN);?

#### SIGPIPE  终止进程 向一个没有读进程的管道写数据