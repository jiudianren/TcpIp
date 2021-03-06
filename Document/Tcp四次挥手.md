
tcp三次握手的过程，accept发生在三次握手哪个阶段？
三次握手：C----->SYN K
              S------>ACK K+1 SYN J
              C------->ACK J+1   
              DONE!
client 的 connect  引起3次握手
server 在socket， bind， listen后，阻塞在accept，三次握手完成后，accept返回一个fd，
因此accept发生在三次握手之后。。。。。。



tcp结束连接怎么握手，time_wait状态是什么,为什么会有time_wait状态？哪一方会有time_wait状态，如何避免time_wait状态占用资源（必须回答的详细）



关于tcp中time_wait状态的4个问题


https://blog.csdn.net/wwh578867817/article/details/46625125


TCP/IP协议就是这样设计的，是不可避免的。主要有两个原因:

1）可靠地实现TCP全双工连接的终止

TCP协议在关闭连接的四次握手过程中，最终的ACK是由主动关闭连接的一端（后面统称A端）发出的，如果这个ACK丢失，对方（后面统称B端）将重发出最终的FIN，因此A端必须维护状态信息（TIME_WAIT）允许它重发最终的ACK。如果A端不维持TIME_WAIT状态，而是处于CLOSED 状态，那么A端将响应RST分节，B端收到后将此分节解释成一个错误（在java中会抛出connection reset的SocketException)。

因而，要实现TCP全双工连接的正常终止，必须处理终止过程中四个分节任何一个分节的丢失情况，主动关闭连接的A端必须维持TIME_WAIT状态 。

 

2）允许老的重复分节在网络中消逝 

TCP分节可能由于路由器异常而“迷途”，在迷途期间，TCP发送端可能因确认超时而重发这个分节，迷途的分节在路由器修复后也会被送到最终目的地，这个迟到的迷途分节到达时可能会引起问题。在关闭“前一个连接”之后，马上又重新建立起一个相同的IP和端口之间的“新连接”，“前一个连接”的迷途重复分组在“前一个连接”终止后到达，而被“新连接”收到了。为了避免这个情况，TCP协议不允许处于TIME_WAIT状态的连接启动一个新的可用连接，因为TIME_WAIT状态持续2MSL，就可以保证当成功建立一个新TCP连接的时候，来自旧连接重复分组已经在网络中消逝。