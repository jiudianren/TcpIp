

##  TCP/IP详解--TCP三次握手建立连接与四次握手终止连接
https://blog.csdn.net/yusiguyuan/article/details/22876185
三次握手耳熟能详，四次挥手估计就，所谓四次挥手（Four-Way Wavehand）即终止TCP连接，就是指断开一个TCP连接时，需要客户端和服务端总共发送4个包以确认连接的断开。


发送FIN,表示 数据发送任务完成，可以继续接受消息。
 由于TCP连接时全双工的，因此，每个方向都必须要单独进行关闭，这一原则是当一方完成数据发送任务后，发送一个FIN来终止这一方向的连接，收到一个FIN只是意味着这一方向上没有数据流动了，即不会再收到数据了，但是在这个TCP连接上仍然能够发送数据，直到这一方向也发送了FIN。首先进行关闭的一方将执行主动关闭，而另一方则执行被动关闭，上图描述的即是如此。


## poll 与 select的区别




## socket用于进程间通讯

child05.c

*socketpai*  机制   
https://blog.csdn.net/longwang155069/article/details/54023186

 socketpair创建的只适用于父子进程或者线程间通信，不能用于两个进程之间通信。如果要实现两个进程之间的双向通信，则需要将socketpair创建的一个描述符fd发送给另一个进程，这相当于两个两个不同的进程访问同一个文件。

dup和dup2的使用方法

https://blog.csdn.net/lurendetiankong/article/details/53487206