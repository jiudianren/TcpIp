
[poll](https://blog.csdn.net/lixungogogo/article/details/52226434)

poll�Ļ�����select���ƣ���select�ڱ�����û�ж���𣬹�����������Ҳ�ǽ�����ѯ��������������״̬���д���
		
		����pollû������ļ����������������ơ� 
����poll��selectͬ������һ��ȱ����ǣ����������ļ������������鱻���帴�����û�̬���ں˵ĵ�ַ�ռ�֮�䣬��������Щ�ļ��������Ƿ���������Ŀ��������ļ����������������Ӷ���������

poll����
����������ʽ������ʾ��

		#include <poll.h>
####int poll ( struct pollfd * fds, unsigned int nfds, int timeout);

��������
struct pollfd * fds
pollfd�ṹ�嶨�����£�

	struct pollfd {
	    int fd;               /* �ļ������� */
	    short events;         /* �ȴ����¼� */
	    short revents;        /* ʵ�ʷ����˵��¼� */
	} ; 

����ÿһ��pollfd�ṹ��ָ����һ�������ӵ��ļ������������Դ��ݶ���ṹ�壬ָʾpoll()���Ӷ���ļ��������� 
����ÿ���ṹ���events���Ǽ��Ӹ��ļ����������¼����룬���û������������ 
����revents�����ļ��������Ĳ�������¼����룬�ں��ڵ��÷���ʱ��������� 
����events����������κ��¼���������revents���з��ء� 
�����Ϸ����¼����£�

	   POLLIN ���������������������ݿɶ���
	����POLLRDNORM ��������  ����ͨ���ݿɶ���
	����POLLRDBAND���������� ���������ݿɶ���
	����POLLPRI���������������� �н������ݿɶ���
	����POLLOUT������������      д���ݲ��ᵼ��������
	����POLLWRNORM����������  д��ͨ���ݲ��ᵼ��������
	����POLLWRBAND����������   д�������ݲ��ᵼ��������
	����POLLMSGSIGPOLL ����������Ϣ���á�

�������⣬revents���л����ܷ��������¼��� 

	   POLLER����   ָ�����ļ���������������
	����POLLHUP���� ָ�����ļ������������¼���
	����POLLNVAL����ָ�����ļ��������Ƿ���

������Щ�¼���events���������壬��Ϊ�����ں��ʵ�ʱ�����ǻ��revents�з��ء�

����ʹ��poll()��select()��һ�����㲻��Ҫ��ʽ�������쳣������档 
����POLLIN | POLLPRI�ȼ���select()�Ķ��¼�. 
����POLLOUT |POLLWRBAND�ȼ���select()��д�¼��� 
����POLLIN�ȼ���POLLRDNORM |POLLRDBAND 
������POLLOUT��ȼ���POLLWRNORM��

�������磬Ҫͬʱ����һ���ļ��������Ƿ�ɶ��Ϳ�д�����ǿ������� eventsΪPOLLIN |POLLOUT����poll����ʱ�����ǿ��Լ��revents�еı�־����Ӧ���ļ������������events�ṹ�塣���POLLIN�¼������ã����ļ����������Ա���ȡ�������������POLLOUT�����ã����ļ�����������д�����������������Щ��־�����ǻ���ģ����ǿ��ܱ�ͬʱ���ã���ʾ����ļ��������Ķ�ȡ��д����������������ض���������

unsigned int nfds,
����nfds_t���͵Ĳ��������ڱ������fds�еĽṹ��Ԫ�ص�������

	int timeout
	����timeout����ָ���ȴ��ĺ�����������I/O�Ƿ�׼���ã�poll���᷵�ء� 
	����timeoutָ��Ϊ����ֵ��ʾ���޳�ʱ��ʹpoll()һֱ����ֱ��һ��ָ���¼����� 
	����timeoutΪ0   �������� �����ȴ� �� ָʾpoll�����������ز��г�׼����I/O���ļ����������������ȴ��������¼�����������£�poll()������������������һ��ѡ�ٳ������������ء�

����ֵ
�����ɹ�ʱ��poll()���ؽṹ����revents��Ϊ0���ļ������������� 
��������ڳ�ʱǰû���κ��¼�������poll()����0�� 
����ʧ��ʱ��poll()����-1��������errnoΪ����ֵ֮һ�� 
����

	����EBADF����       һ�������ṹ����ָ�����ļ���������Ч��
	����EFAULTfds���� ָ��ָ��ĵ�ַ�������̵ĵ�ַ�ռ䡣
	����EINTR��������  ������¼�֮ǰ����һ���źţ����ÿ������·���
	����EINVALnfds������������PLIMIT_NOFILEֵ��
	����ENOMEM����     �����ڴ治�㣬�޷��������



#�ܽ�
����poll�����Ϻ�selectû�����������û���������鿽�����ں˿ռ䣬Ȼ���ѯÿ��fd��Ӧ���豸״̬������豸���������豸�ȴ������м���һ�����������
�������������fd��û�з��־����豸�������ǰ���̣�ֱ���豸��������������ʱ�������Ѻ�����Ҫ�ٴα���fd�� 
����
   ������̾����˶����ν�ı����� 

	poll����һ���ص��ǡ�ˮƽ�����������������fd��û�б�������ô�´�pollʱ���ٴα����fd�� 
����

poll��select�Ĳ�ͬ��ͨ��һ��pollfd�������ں˴�����Ҫ��ע���¼�����û�����������������ƣ�
pollfd�е�events�ֶκ�revents�ֱ����ڱ�ʾ��ע���¼��ͷ������¼���

	��pollfd����ֻ��Ҫ����ʼ��һ��
 
����poll��ʵ�ֻ�����select���ƣ����Ӧ�ں��е�sys_poll��ֻ����poll���ں˴���pollfd���飬Ȼ���pollfd�е�ÿ������������poll����ȴ���fdset��˵��pollЧ�ʸ��ߡ�poll���غ���Ҫ��pollfd�е�ÿ��Ԫ�ؼ����reventsֵ������ָ�¼��Ƿ�����

�ŵ�

	1��poll() ��Ҫ�󿪷��߼�������ļ���������һ�Ĵ�С�� 
	2��poll() ��Ӧ������Ŀ���ļ���������ʱ���ٶȸ��죬�����select�� 
	3����û����������������ƣ�ԭ�������ǻ����������洢�ġ�

ȱ��

	1��������fd�����鱻���帴�����û�̬���ں˵�ַ�ռ�֮�䣬�����������ĸ����ǲ��������塣 
	2����selectһ����poll���غ���Ҫ��ѯpollfd����ȡ������������

����poll�ķ���������

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
