epoll
���� ��linux���������У��ܳ���ʱ�䶼��ʹ��select�����¼�������
     ��linux�µ��ں��У�����һ���滻���Ļ��ƣ�����epoll�� 
����
   �����select��
   
    epoll���ĺô��������������ż���fd��Ŀ������������Ч�ʡ�
   
  ��Ϊ���ں��е�selectʵ���У����ǲ�����ѯ������ģ���ѯ��fd��ĿԽ�࣬��Ȼ��ʱԽ�ࡣ 
�����select��poll��˵��epoll������û�����������ơ� 


    epollʹ��һ���ļ������������������������û���ϵ���ļ����������¼���ŵ��ں˵�һ���¼����У��������û��ռ���ں˿ռ��copyֻ��һ�Ρ�

epoll�ӿ�
   epoll����������Ҫ�����ӿڣ��ֱ����£�

	#include <sys/epoll.h>
	int epoll_create(int size);
	int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
	int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);

����
����Ҫ����epoll_create����һ��epoll���󡣲���size���ں˱�֤�ܹ���ȷ��������������������������ʱ�ں˿ɲ���֤Ч���� 
����epoll_ctl���Բ������潨����epoll�����磬���ս�����socket���뵽epoll�������أ����߰� epoll���ڼ�ص�ĳ��socket����Ƴ�epoll�����ټ�����ȵȡ� 
����epoll_wait�ڵ���ʱ���ڸ�����timeoutʱ���ڣ����ڼ�ص����о�������¼�����ʱ���ͷ����û�̬�Ľ��̡�

### int epoll_create(int size);
��������һ��epoll�ľ����size���������ں������������Ŀһ���ж�����������ͬ��select()�еĵ�һ��������������������fd+1��ֵ�� 
������Ҫע����ǣ���������epoll����������ǻ�ռ��һ��fdֵ����linux������鿴/proc/����id/fd/�����ܹ��������fd�ģ�������ʹ����epoll�󣬱������close()�رգ�������ܵ���fd���ľ���

### int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
����epoll���¼�ע�ắ��������ͬ��select()���ڼ����¼�ʱ�����ں�Ҫ����ʲô���͵��¼�epoll���¼�ע�ắ��. 
��������ͬ��select()���ڼ����¼�ʱ�����ں�Ҫ����ʲô���͵��¼���������������ע��Ҫ�������¼����͡� 
������һ��������epoll_create()�ķ���ֵ���ڶ���������ʾ������������������ʾ��

	1 EPOLL_CTL_ADD��ע���µ�fd��epfd�У�
	2 EPOLL_CTL_MOD���޸��Ѿ�ע���fd�ļ����¼���
	3 EPOLL_CTL_DEL����epfd��ɾ��һ��fd��

������������������Ҫ������fd�� 
�������ĸ������Ǹ����ں���Ҫ����ʲô�£�struct epoll_event�ṹ���£�

	struct epoll_event {
	  __uint32_t events;  /* Epoll events */
	  epoll_data_t data;  /* User data variable */
	};

events���������¼�����ļ��ϣ�

	EPOLLIN ����ʾ��Ӧ���ļ����������Զ��������Զ�SOCKET�����رգ���
	EPOLLOUT����ʾ��Ӧ���ļ�����������д��
	EPOLLPRI����ʾ��Ӧ���ļ��������н��������ݿɶ�������Ӧ�ñ�ʾ�д������ݵ�������
	EPOLLERR����ʾ��Ӧ���ļ���������������
	EPOLLHUP����ʾ��Ӧ���ļ����������Ҷϣ�
	EPOLLET�� ��EPOLL��Ϊ��Ե����(Edge Triggered)ģʽ�����������ˮƽ����(Level Triggered)��˵�ġ�
	EPOLLONESHOT��ֻ����һ���¼���������������¼�֮���������Ҫ�����������socket�Ļ�����Ҫ�ٴΰ����socket���뵽EPOLL������

### int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
�����ȴ��¼��Ĳ�����������select()���á� 
��������events�������ں˵õ��¼��ļ��ϣ�maxevents��֮�ں����events�ж�����maxevents��ֵ���ܴ��ڴ���epoll_create()ʱ��size�� 
��������timeout�ǳ�ʱʱ�䣨���룬0���������أ�-1����ȷ����Ҳ��˵��˵���������������ú���������Ҫ������¼���Ŀ���緵��0��ʾ�ѳ�ʱ�� 
����epoll_wait��Χ֮��Ӧ����һ��ѭ�����������е��¼��� 
�������ǵ���epoll_ waitʱ���൱����������select/poll��������ʱȴ���ô���socket������ںˣ���Ϊ�ں��Ѿ���epoll_ctl���õ���Ҫ��صľ���б� 
�������ԣ�ʵ�����������epoll_ create���ں˾��Ѿ����ں�̬��ʼ׼������洢Ҫ��صľ���ˣ�ÿ�ε���epoll_ctlֻ�������ں˵����ݽṹ�������µ�socket����� 
�������ں��һ�н��ļ������ԣ�epoll���ں�ע����һ���ļ�ϵͳ�����ڴ洢�����ı����socket���������epoll_createʱ���ͻ�����������epoll�ļ�ϵͳ�ﴴ��һ��file��㡣��Ȼ���file������ͨ�ļ�����ֻ������epoll��

# epollʵ�ֻ���
����epoll�ڱ��ں˳�ʼ��ʱ������ϵͳ��������ͬʱ�Ὺ�ٳ�epoll�Լ����ں˸���cache�������ڰ���ÿһ���������ص�socket�� 
������Щsocket���Ժ��������ʽ�������ں�cache���֧�ֿ��ٵĲ��ҡ����롢ɾ���� 
��������ں˸���cache�������ǽ��������������ڴ�ҳ��Ȼ����֮�Ͻ���slab�㡣 
�����򵥵�˵�����������Ϸ��������Ҫ��size���ڴ����ÿ��ʹ��ʱ����ʹ�ÿ��е��ѷ���õĶ��� 
����epoll�ĸ�Ч�����ڣ������ǵ���epoll_ctl����������ʮ��(10k,100k�����������ԣ���1000k(�������ﲻ��))�����ʱ��epoll_ wait��Ȼ���Էɿ�ķ��أ�����Ч�Ľ������¼��ľ���������û��� 
�����������������ڵ���epoll_createʱ���ں˳��˰�������epoll�ļ�ϵͳ�ｨ�˸�file��㣬���ں�cache�ｨ�˸���������ڴ洢�Ժ�epoll_ctl������socket�⣬�����ٽ���һ��list�������ڴ洢׼���������¼�. 
������epoll_ wait����ʱ�������۲����list��������û�����ݼ��ɡ������ݾͷ��أ�û�����ݾ�sleep���ȵ�timeoutʱ�䵽��ʹ����û����Ҳ���ء����ԣ�epoll_wait�ǳ���Ч�� 
�������ң�ͨ������¼�ʹ����Ҫ��ذ���Ƶľ�������һ��Ҳֻ���غ�������׼������������ѣ����ԣ�epoll_wait����Ҫ���ں�̬copy�����ľ�����û�̬���ѡ� 
������ô�����׼������list��������ôά�����أ� 
����������ִ��epoll_ctlʱ�����˰�socket�ŵ�epoll�ļ�ϵͳ��file�����Ӧ�ĺ������֮�⣬������ں��жϴ������ע��һ���ص������������ںˣ�������������жϵ��ˣ��Ͱ����ŵ�׼������list����� 
�������ԣ���һ��socket�������ݵ��ˣ��ں��ڰ������ϵ�����copy���ں��к������socket���뵽׼�������������ˡ�
������ˣ�һ�ź������һ��׼��������������������ں�cache���Ͱ����ǽ���˴󲢷��µ�socket�������⡣ 
����ִ��epoll_ createʱ�������˺�����;�������ִ��epoll_ctlʱ���������socket����������ں�������Ƿ���ڣ������������أ�����������ӵ������ϣ�Ȼ�����ں�ע��ص����������ڵ��ж��¼�����ʱ��׼�����������в������ݡ�ִ��epoll_waitʱ���̷���׼����������������ݼ��ɡ�

����ģʽ
����epoll���ļ��������Ĳ���������ģʽ��LT��level trigger����ET��edge trigger����LTģʽ��Ĭ��ģʽ��LTģʽ��ETģʽ���������£�

LTģʽ��
������epoll_ wait��⵽�������¼������������¼�֪ͨӦ�ó���Ӧ�ó�����Բ�����������¼����´ε���epoll_waitʱ�����ٴ���ӦӦ�ó���֪ͨ���¼���

ETģʽ��
������epoll_ wait��⵽�������¼������������¼�֪ͨӦ�ó���Ӧ�ó����������������¼�������������´ε���epoll_waitʱ�������ٴ���ӦӦ�ó���֪ͨ���¼���

����ETģʽ�ںܴ�̶��ϼ�����epoll�¼����ظ������Ĵ��������Ч��Ҫ��LTģʽ�ߡ� 
����epoll������ETģʽ��ʱ�򣬱���ʹ�÷������׽ӿڣ��Ա�������һ���ļ������������/����д�����Ѵ������ļ������������������ 
������ôETģʽ����ô�������أ�

ETģʽ��ԭ��
������һ��socket��������¼�ʱ���ں˻�Ѹþ������������˵��׼������list������ʱ���ǵ���epoll_ wait�����׼��������socket�������û�̬�ڴ棬Ȼ�����׼������list���� 
�������epoll_ wait�����Щsocket���������ETģʽ������LTģʽ�ľ���ˣ���������Щsocket��ȷʵ��δ������¼�ʱ���ְѸþ���Żص��ո���յ�׼�����������ˡ� 
�������ԣ���ET�ľ����ֻҪ�����滹���¼���epoll_ waitÿ�ζ��᷵�ء���ETģʽ�ľ�������������жϵ�����ʹsocket�ϵ��¼�û�д����꣬Ҳ�ǲ���δδ�epoll_wait���صġ�

�ŵ�
IO��·����֮select�����ܽ� 
IO��·����֮poll�����ܽ� 
����ǰ��ƪ����������˵��select��poll��ȱ�㣬��������̸̸epoll���ŵ㣺

1��֧��һ�����̴򿪴���Ŀ��socket������(FD)

����select������ܵ���һ���������򿪵�FD����һ�����Ƶģ���FD_SETSIZE���ã�Ĭ��ֵ��1024/2048��������Щ��Ҫ֧�ֵ�����������Ŀ��IM��������˵��Ȼ̫���ˡ���ʱ����һ�ǿ���ѡ���޸������Ȼ�����±����ںˡ����� epoll��û��������ƣ�����֧�ֵ�FD�����������Դ��ļ�����Ŀ���������һ��Զ����2048,�ٸ�����,��1GB�ڴ�Ļ����ϴ�Լ��10�����ң�������Ŀ����cat /proc/sys/fs/file-max�쿴,һ����˵�����Ŀ��ϵͳ�ڴ��ϵ�ܴ�

2��IOЧ�ʲ���FD��Ŀ���Ӷ������½�

������ͳ��select/poll��һ������������ǵ���ӵ��һ���ܴ��socket���ϣ���������������ʱ����һʱ��ֻ�в��ֵ�socket�ǡ���Ծ���ģ�����select/pollÿ�ε��ö�������ɨ��ȫ���ļ��ϣ�����Ч�ʳ��������½�������epoll������������⣬��ֻ��ԡ���Ծ����socket���в�����������Ϊ���ں�ʵ����epoll�Ǹ���ÿ��fd�����callback����ʵ�ֵġ���ô��ֻ�С���Ծ����socket�Ż�������ȥ���� callback����������idle״̬socket�򲻻ᣬ������ϣ�epollʵ����һ����α��AIO����Ϊ��ʱ���ƶ�����Linux�ںˡ� 
3��ʹ��mmap�����ں����û��ռ����Ϣ���ݡ�

�������ʵ�����漰��epoll�ľ���ʵ���ˡ�������select,poll����epoll����Ҫ�ں˰�FD��Ϣ֪ͨ���û��ռ䣬��α��ⲻ��Ҫ���ڴ濽���ͺ���Ҫ��������ϣ�epoll��ͨ���ں����û��ռ�mmapͬһ���ڴ�ʵ�ֵġ�



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
	
	    /* ����ÿ����������򿪵�����ļ��� */
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
	    /* ���� epoll ������Ѽ��� socket ���뵽 epoll ������ */
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
	        /* �ȴ����¼����� */
	        nfds = epoll_wait(kdpfd, events, curfds, -1);
	        if (nfds == -1)
	        {
	            perror("epoll_wait");
	            continue;
	        }
	        /* ���������¼� */
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
	            // ����ͻ�������
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
	    nread = read(connfd, buf, MAXLINE);//��ȡ�ͻ���socket��
	
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
	    write(connfd, buf, nread);//��Ӧ�ͻ���  
	    return 0;
	}