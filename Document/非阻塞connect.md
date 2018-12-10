[非阻塞方式connect编程](https://www.cnblogs.com/cz-blog/p/4530641.html  "ss")


说明：由于程序用select等待连接完成，可以设置一个select等待时间限制，从而缩短connect超时时间。  
多数实现中，connect的超时时间在75秒到几分钟之间。  
有时程序希望在等待一定时间内结束，使用非阻塞connect可以防止阻塞75秒，在多线程网络编程中，尤其必要。  
 
 例如有一个通过建立线程与其他主机进行socket通信的应用程序，如果建立的线程使用阻塞connect与远程通信，  
 当有几百个线程并发的时候，由于网络延迟而全部阻塞，阻塞的线程不会释放系统的资源，   
 同一时刻阻塞线程超过一定数量时候，系统就不再允许建立新的线程（每个进程由于进程空间的原因能产生的线程有限），  
 如果使用非阻塞的connect，连接失败使用select等待很短时间，如果还没有连接后，  
 线程立刻结束释放资源，防止大量线程阻塞而使程序崩溃。
	
	int tcp_connect(char *host, int port)
	{
	    int sock, flags;
	    struct sockaddr_in rsock;
	    struct hostent * hostinfo;
	    struct in_addr * addp;
	    struct linger stLinger = { 1, 2 }; 
	    
	    memset ((char *)&rsock,0,sizeof(rsock));
	    
	    if ((hostinfo = gethostbyname(host)) == NULL)
	    {
	        return -1;
	    }
	    //步骤一:socket
	    sock = socket(AF_INET, SOCK_STREAM, 0);
	    if (sock == -1)
	    {
	        return -1;
	    }
	    //步骤二:填充
	    addp = (struct in_addr *)*(hostinfo->h_addr_list);
	    rsock.sin_addr = *addp;
	    rsock.sin_family = AF_INET;
	    rsock.sin_port = htons(port);
	    
	    int ret = 0, error = -1, slen = sizeof(int);
	    timeval tm;
	    fd_set set;
	    
	    unsigned long ul = 1;
	    ioctl(sock, FIONBIO, &ul); //设置为非阻塞模式
	    //步骤三:connect，此时socket设置为非阻塞，connect调用后，无论连接是否建立立即返回-1，
	
	    if (connect(sock, (struct sockaddr *)(&rsock), sizeof(rsock)) == -1)
	    {
	    //表示此时tcp三次握手仍旧进行，如果errno不是EINPROGRESS，则说明连接错误，程序结束
	        if (errno != EINPROGRESS)
	        {
	            ret = 0;
	        }
	        else
	        {
	        
	            tm.tv_sec  = 5;
	            tm.tv_usec = 0;
	            FD_ZERO(&set);
	            FD_SET(sock, &set);
	            //监听写集合
	            if (select(sock+1, NULL, &set, NULL, &tm) > 0)
	            {
	            //过调用 getsockopt(sockfd,SOL_SOCKET,SO_ERROR,&error,&len); 函数返回值来判断是否发生错误
	            //rror返回0则表示连接成功！
	                getsockopt(sock, SOL_SOCKET, SO_ERROR, &error, (socklen_t *)&slen);
	                if (error == 0)
	                {
	                    ret = 1;
	                }
	                else
	                {
	                    ret = 0;
	                }
	            }
	            else
	            {
	                ret = 0;
	            }
	        }
	    }
	    else
	    {//客户端和服务器已经建立连接
	        ret = 1;
	    }
	    
	    ul = 0;
	    ioctl(sock, FIONBIO, &ul); //设置为阻塞模式
	    //ret =1:表示正常建立连接
	    if (!ret)
	    {
	        close(sock);
	        return -1;
	    }
	    //linger :徘徊的意思。SO_LINGER:表示经历time_wait阶段，且时间是stLinger中第二个参数指定的值。
	    flags = setsockopt(sock, SOL_SOCKET, SO_LINGER, &stLinger, sizeof(struct linger));
	    if (flags == -1)
	    {
	        close(sock);
	        return -1;
	    }
	    
	    return sock;
	}