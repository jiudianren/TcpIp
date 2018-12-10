[��������ʽconnect���](https://www.cnblogs.com/cz-blog/p/4530641.html  "ss")


˵�������ڳ�����select�ȴ�������ɣ���������һ��select�ȴ�ʱ�����ƣ��Ӷ�����connect��ʱʱ�䡣  
����ʵ���У�connect�ĳ�ʱʱ����75�뵽������֮�䡣  
��ʱ����ϣ���ڵȴ�һ��ʱ���ڽ�����ʹ�÷�����connect���Է�ֹ����75�룬�ڶ��߳��������У������Ҫ��  
 
 ������һ��ͨ�������߳���������������socketͨ�ŵ�Ӧ�ó�������������߳�ʹ������connect��Զ��ͨ�ţ�  
 ���м��ٸ��̲߳�����ʱ�����������ӳٶ�ȫ���������������̲߳����ͷ�ϵͳ����Դ��   
 ͬһʱ�������̳߳���һ������ʱ��ϵͳ�Ͳ����������µ��̣߳�ÿ���������ڽ��̿ռ��ԭ���ܲ������߳����ޣ���  
 ���ʹ�÷�������connect������ʧ��ʹ��select�ȴ��ܶ�ʱ�䣬�����û�����Ӻ�  
 �߳����̽����ͷ���Դ����ֹ�����߳�������ʹ���������
	
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
	    //����һ:socket
	    sock = socket(AF_INET, SOCK_STREAM, 0);
	    if (sock == -1)
	    {
	        return -1;
	    }
	    //�����:���
	    addp = (struct in_addr *)*(hostinfo->h_addr_list);
	    rsock.sin_addr = *addp;
	    rsock.sin_family = AF_INET;
	    rsock.sin_port = htons(port);
	    
	    int ret = 0, error = -1, slen = sizeof(int);
	    timeval tm;
	    fd_set set;
	    
	    unsigned long ul = 1;
	    ioctl(sock, FIONBIO, &ul); //����Ϊ������ģʽ
	    //������:connect����ʱsocket����Ϊ��������connect���ú����������Ƿ�����������-1��
	
	    if (connect(sock, (struct sockaddr *)(&rsock), sizeof(rsock)) == -1)
	    {
	    //��ʾ��ʱtcp���������Ծɽ��У����errno����EINPROGRESS����˵�����Ӵ��󣬳������
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
	            //����д����
	            if (select(sock+1, NULL, &set, NULL, &tm) > 0)
	            {
	            //������ getsockopt(sockfd,SOL_SOCKET,SO_ERROR,&error,&len); ��������ֵ���ж��Ƿ�������
	            //rror����0���ʾ���ӳɹ���
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
	    {//�ͻ��˺ͷ������Ѿ���������
	        ret = 1;
	    }
	    
	    ul = 0;
	    ioctl(sock, FIONBIO, &ul); //����Ϊ����ģʽ
	    //ret =1:��ʾ������������
	    if (!ret)
	    {
	        close(sock);
	        return -1;
	    }
	    //linger :�ǻ�����˼��SO_LINGER:��ʾ����time_wait�׶Σ���ʱ����stLinger�еڶ�������ָ����ֵ��
	    flags = setsockopt(sock, SOL_SOCKET, SO_LINGER, &stLinger, sizeof(struct linger));
	    if (flags == -1)
	    {
	        close(sock);
	        return -1;
	    }
	    
	    return sock;
	}