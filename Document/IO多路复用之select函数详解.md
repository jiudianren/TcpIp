
https://blog.csdn.net/lixungogogo/article/details/52219951

IO����
��������������������������̵�ģ�ͣ��ͻ��˷������������˻����һ��������������з���ÿ����һ���ͻ�����Ͳ���һ������������Ȼ�����̲����������ƵĲ��������Ϊ�˽�������ͻ��˷��ʵ����⣬������IO���ü����� 
��������һ�����̿���ͬʱ�Զ���ͻ�������з���

����Ҳ����˵IO���õġ����ʡ��ǽ���(׼ȷ��˵���õ���select��poll����Ϊ����Ҳ�ǿ�����select��poll��ʵ�ֵ�)������һ������(select��poll)���Զ��IO���з�����Ȼ�ͻ��˷�����IO�ǲ����ĵ���IO����Ķ�д���ݶ����������û��׼���õģ���˾Ϳ�������һ������(select��poll)������IO�������Щ���ݵ�״̬��һ��IO�����ݿ��Խ��ж�д�ˣ����̾�����������IO���з��� 
����IO��·����ָ�ں�һ�����ֽ���ָ����һ�����߶��IO����׼����ȡ������֪ͨ�ý��̡� 
����IO��·�����������³��ϣ� 
����1.���ͻ�������������ʱ��һ���ǽ���ʽ����������׽ӿڣ�������ʹ��I/O���á� 
����2.��һ���ͻ�ͬʱ�������׽ӿ�ʱ������������ǿ��ܵģ������ٳ��֡� 
����3.���һ��TCP��������Ҫ��������׽ӿڣ���Ҫ�����������׽ӿڣ�һ��ҲҪ�õ�I/O���á� 
����4.���һ����������Ҫ����TCP����Ҫ����UDP��һ��Ҫʹ��I/O���á� 
����5.���һ��������Ҫ�������������Э�飬һ��Ҫʹ��I/O���á�

���������̺Ͷ��̼߳�����ȣ�I/O��·���ü��������������ϵͳ����С��ϵͳ���ش�������/�̣߳�Ҳ����ά����Щ����/�̣߳��Ӷ�����С��ϵͳ�Ŀ�����

select����
�����ú����������ָʾ�ں˵ȴ�����¼��е��κ�һ����������ֻ����һ������ʱ�䷢������һ��ָ����ʱ���Ż������� 
����

#include <sys/select.h>
#include >sys/time.h>

int select(int maxfdp1,fd_set *readset,fd_set *writeset,fd_set *exceptset,const struct timeval *timeout);
1
2
3
4
����ֵ��
�������о�����������������Ŀ������ʱ��Ϊ0����������Ϊ-1

����
��һ����������int maxfdp1
������һ������maxfdp1ָ�������Ե������ָ����� 
��������ֵ�Ǵ����Ե���������ּ�1����˰Ѹò�������Ϊmaxfdp1����������0��1��2��maxfdp1-1���������ԡ� 
��Ϊ�ļ��������Ǵ�0��ʼ�ġ�

fd_set *readset
fd_set *writeset
fd_set *exceptset
�����м����������readset��writeset��exceptsetָ������Ҫ���ں˲��Զ���д���쳣�����������֡��� 
���������ĳһ��������������Ȥ���Ϳ��԰�����Ϊ��ָ�롣struct fd_set�������Ϊһ�����ϣ���������д�ŵ����ļ�����������ͨ�������ĸ���������ã�

void FD_ZERO(fd_set *fdset);           
//��ռ���

void FD_SET(int fd, fd_set *fdset);   
//��һ���������ļ����������뼯��֮��

void FD_CLR(int fd, fd_set *fdset);   
//��һ���������ļ��������Ӽ�����ɾ��

int FD_ISSET(int fd, fd_set *fdset);   
// ��鼯����ָ�����ļ��������Ƿ���Զ�д 

	const struct timeval *timeout
	����timeout��֪�ں˵ȴ���ָ���������е��κ�һ�������ɻ�����ʱ�䡣��timeval�ṹ����ָ�����ʱ���������΢������
	
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
������������ֿ��ܣ� 
������Զ�ȴ���ȥ��������һ��������׼����I/Oʱ�ŷ��ء�Ϊ�ˣ��Ѹò�������Ϊ��ָ��NULL�� 
�����ȴ�һ�ι̶�ʱ�䣺����һ��������׼����I/Oʱ���أ����ǲ������ɸò�����ָ���timeval�ṹ��ָ����������΢������ 
3.�������ȴ�����������ֺ��������أ����Ϊ��ѯ��Ϊ�ˣ��ò�������ָ��һ��timeval�ṹ���������еĶ�ʱ��ֵ����Ϊ0��

select�����ĵ��ù���
��1��ʹ��copy_from_user���û��ռ俽��fd_set���ں˿ռ�

��2��ע��ص�����__pollwait

��3����������fd

�����������Ӧ��poll����������socket�����poll������sock_poll��sock_poll�����������õ�tcp_poll,udp_poll����datagram_poll��

��4����tcp_pollΪ���������ʵ�־���__pollwait��Ҳ��������ע��Ļص�������

��5��__pollwait����Ҫ�������ǰ�current����ǰ���̣��ҵ��豸�ĵȴ������У���ͬ���豸�в�ͬ�ĵȴ����У�����tcp_poll��˵����ȴ�������sk->sk_sleep��ע��ѽ��̹ҵ��ȴ������в�����������Ѿ�˯���ˣ������豸�յ�һ����Ϣ�������豸������д���ļ����ݣ������豸���󣬻ỽ���豸�ȴ�������˯�ߵĽ��̣���ʱcurrent�㱻�����ˡ�

��6��poll��������ʱ�᷵��һ��������д�����Ƿ������mask���룬�������mask�����fd_set��ֵ��

��7��������������е�fd����û�з���һ���ɶ�д��mask���룬������schedule_timeout�ǵ���select�Ľ��̣�Ҳ����current������˯�ߡ����豸��������������Դ�ɶ�д�󣬻ỽ����ȴ�������˯�ߵĽ��̡��������һ���ĳ�ʱʱ�䣨schedule_timeoutָ����������û�˻��ѣ������select�Ľ��̻����±����ѻ��CPU���������±���fd���ж���û�о�����fd��

��8����fd_set���ں˿ռ俽�����û��ռ䡣

select˯�ߺͻ��ѹ���
����select��������õȴ����л������û������ʵ���û����Դ�ɶ�/дʱ˯�ߣ�����Դ�ɶ�/дʱ���ѡ�

select˯�߹���
����select��ѭ��������������fd_��set�ڵ������ļ���������Ӧ�����������poll������ 
�������������ṩ��poll�������ȻὫ����select���û����̲��뵽���豸������Ӧ��Դ�ĵȴ�����(���/д�ȴ�����)��Ȼ�󷵻�һ��bitmask����select��ǰ��Դ��Щ���á��� 
������selectѭ������������fd_set��ָ�����ļ���������Ӧ��poll���������û��һ����Դ����(��û��һ���ļ��ɹ�����)����select�øý���˯�ߣ�һֱ�ȵ�����Դ����Ϊֹ�����̱�����(����timeout)��������ִ�С�

select���ѹ���
�������Ѹý��̵Ĺ���ͨ������������ļ����豸������ʵ�ֵġ� 
������������ά�������������Դ��д�ĵȴ����С����豸��������������Դ��Ϊ�ɶ�д�����н���˯���ڸ���Դ�ĵȴ�������ʱ���ͻỽ�������Դ�ȴ������ϵĽ��̡�

# select��ȱ��
������1.ÿ�ε���select������Ҫ��fd���ϴ��û�̬�������ں�̬�����������fd�ܶ�ʱ��ܴ� 
����2.ͬʱÿ�ε���select����Ҫ���ں˱������ݽ���������fd�����������fd�ܶ�ʱҲ�ܴ� 
����3.select֧�ֵ��ļ�����������̫С�ˣ�Ĭ����1024


ʹ��select����д�ķ������������£�

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
