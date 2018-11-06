浅谈Http长连接和Keep-Alive以及Tcp的Keepalive

Keep-Alive模式： 
我们知道Http协议采用“请求-应答”模式，当使用普通模式，即非Keep-Alive模式时，每个请求/应答，客户端和服务器都要新建一个连接，完成之后立即断开连接；当使用Keep-Alive模式时，Keep-Alive功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接。 
http 1.0中默认是关闭的，需要在http头加入”Connection: Keep-Alive”，才能启用Keep-Alive； 
http 1.1中默认启用Keep-Alive，如果加入”Connection: close “才关闭。目前大部分浏览器都是用http1.1协议，也就是说默认都会发起Keep-Alive的连接请求了，所以是否能完成一个完整的Keep- Alive连接就看服务器设置情况。下图是普通模式和长连接模式的请求对比：  


开启Keep-Alive的优缺点： 
优点：Keep-Alive模式更加高效，因为避免了连接建立和释放的开销。 
缺点：长时间的Tcp连接容易导致系统资源无效占用，浪费系统资源。

当保持长连接时，如何判断一次请求已经完成？ 
Content-Length 
Content-Length表示实体内容的长度。浏览器通过这个字段来判断当前请求的数据是否已经全部接收。  
所以，当浏览器请求的是一个静态资源时，即服务器能明确知道返回内容的长度时，可以设置Content-Length来控制请求的结束。但当服务器并不知道请求结果的长度时，如一个动态的页面或者数据，Content-Length就无法解决上面的问题，这个时候就需要用到Transfer-Encoding字段。

Transfer-Encoding 
Transfer-Encoding是指传输编码，在上面的问题中，当服务端无法知道实体内容的长度时，就可以通过指定Transfer-Encoding: chunked来告知浏览器当前的编码是将数据分成一块一块传递的。当然, 还可以指定Transfer-Encoding: gzip, chunked表明实体内容不仅是gzip压缩的，还是分块传递的。最后，当浏览器接收到一个长度为0的chunked时， 知道当前请求内容已全部接收。

Keep-Alive timeout： 
Httpd守护进程，一般都提供了keep-alive timeout时间设置参数。比如nginx的keepalive_timeout，和Apache的KeepAliveTimeout。这个keepalive_timout时间值意味着：一个http产生的tcp连接在传送完最后一个响应后，还需要hold住keepalive_timeout秒后，才开始关闭这个连接。 
当httpd守护进程发送完一个响应后，理应马上主动关闭相应的tcp连接，设置 keepalive_timeout后，httpd守护进程会想说：”再等等吧，看看浏览器还有没有请求过来”，这一等，便是keepalive_timeout时间。如果守护进程在这个等待的时间里，一直没有收到浏览器发过来http请求，则关闭这个http连接。

Tcp的Keepalive： 
连接建立之后，如果客户端一直不发送数据，或者隔很长时间才发送一次数据，当连接很久没有数据报文传输时如何去确定对方还在线，到底是掉线了还是确实没有数据传输，连接还需不需要保持，这种情况在TCP协议设计中是需要考虑到的。 
TCP协议通过一种巧妙的方式去解决这个问题，当超过一段时间之后，TCP自动发送一个数据为空的报文（侦测包）给对方，如果对方回应了这个报文，说明对方还在线，连接可以继续保持，如果对方没有报文返回，并且重试了多次之后则认为链接丢失，没有必要保持连接。

tcp keep-alive是TCP的一种检测TCP连接状况的保鲜机制。tcp keep-alive保鲜定时器，支持三个系统内核配置参数： 
net.ipv4.tcp_keepalive_intvl = 15 
net.ipv4.tcp_keepalive_probes = 5 
net.ipv4.tcp_keepalive_time = 1800 
keepalive是TCP保鲜定时器，当网络两端建立了TCP连接之后，闲置（双方没有任何数据流发送往来）了tcp_keepalive_time后，服务器就会尝试向客户端发送侦测包，来判断TCP连接状况(有可能客户端崩溃、强制关闭了应用、主机不可达等等)。如果没有收到对方的回答(ack包)，则会在 tcp_keepalive_intvl后再次尝试发送侦测包，直到收到对方的ack,如果一直没有收到对方的ack,一共会尝试 tcp_keepalive_probes次，每次的间隔时间在这里分别是15s, 30s, 45s, 60s, 75s。如果尝试tcp_keepalive_probes,依然没有收到对方的ack包，则会丢弃该TCP连接。TCP连接默认闲置时间是2小时，一般设置为30分钟足够了

---------------------

本文来自 许文杰 的CSDN 博客 ，全文地址请点击：https://blog.csdn.net/weixin_37672169/article/details/80283935?utm_source=copy 