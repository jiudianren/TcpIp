linux������֮shutdown() �� close()�������
https://blog.csdn.net/lgp88/article/details/7176509

1.close()����

	#include<unistd.h>
	int close(int sockfd);     //���سɹ�Ϊ0������Ϊ-1.
	
close һ���׽��ֵ�Ĭ����Ϊ�ǰ��׽��ֱ��Ϊ�ѹرգ�Ȼ���������ص����ý��̣����׽����������������ɵ��ý���ʹ�ã�
Ҳ����˵����������Ϊread��write�ĵ�һ��������Ȼ��TCP�����Է������Ŷӵȴ����͵��Զ˵��κ����ݣ�������Ϻ�������������TCP������ֹ���С�

 �ڶ���̲����������У����ӽ��̹������׽��֣��׽������������ü�����¼�Ź����ŵĽ��̸������������̻�ĳһ�ӽ���close���׽���ʱ��
 ���������ü�������Ӧ�ļ�һ�������ü����Դ�����ʱ�����close���þͲ�������TCP����·���ֶ������̡�

2.shutdown()����

	#include<sys/socket.h>
	int shutdown(int sockfd,int howto);  //���سɹ�Ϊ0������Ϊ-1.
	
 �ú�������Ϊ������howto��ֵ

	 1.SHUT_RD��ֵΪ0���ر����ӵĶ���һ�롣
	 2.SHUT_WR��ֵΪ1���ر����ӵ�д��һ�롣
	 3.SHUT_RDWR��ֵΪ2�����ӵĶ���д���رա�

  ��ֹ�������ӵ�ͨ�÷����ǵ���close��������ʹ��shutdown�ܸ��õĿ��ƶ������̣�ʹ�õڶ�����������

3.������������

close��shutdown��������Ҫ�����ڣ�

close������ر��׽���ID������������Ľ��̹���������׽��֣���ô����Ȼ�Ǵ򿪵ģ����������Ȼ������������д��
������ʱ�����Ƿǳ���Ҫ��?���ر��Ƕ��ڶ���̲�����������˵��

 ��shutdown���жϽ��̹�����׽��ֵ��������ӣ���������׽��ֵ����ü����Ƿ�Ϊ�㣬
 ��Щ��ͼ���ý��̽�����յ�EOF��ʶ����Щ��ͼд�Ľ��̽����⵽SIGPIPE�źţ�
 ͬʱ������shutdown�ĵڶ�������ѡ������ķ�ʽ��


 ���潫չʾһ���ͻ�������Ƭ����˵��ʹ��close��shutdown�������Ĳ�ͬ�����

�ͻ������������̣������̺��ӽ��̣��ӽ������ڸ����̺ͷ���������֮��fork�����ģ��ӽ��̷��ͱ�׼�����ն˼����������ݵ��������ˣ�֪�����յ�EOF��ʶ����������������Է������˵���Ӧ���ݡ�


	   /* First  Sample client fragment,
	    * ����Ĵ��뼰��������������       */
	      s=connect(...);
	      if( fork() ){   /*      The child, it copies its stdin to the socket              */
	          while( gets(buffer) >0)
	              write(s,buf,strlen(buffer));
	              close(s);
	              exit(0);
	      }
	      else {          /* The parent, it receives answers  */
	           while( (n=read(s,buffer,sizeof(buffer)){
	               do_something(n,buffer);
	               /* Connection break from the server is assumed  */
	               /* ATTENTION: deadlock here                     */
	            wait(0); /* Wait for the child to exit          */
	            exit(0);
	       }
 

������δ��룬���������������ӽ��̻�ȡ���׼�ն˵����ݣ�д���׽��ֺ�close�׽��֣����˳����������˽��������ݼ�⵽EOF����ʾ�����ѷ����꣩��Ҳ�ر����ӣ����˳������Ÿ����̶�ȡ�����������Ӧ�����ݣ����˳���Ȼ������ʵ���������ӵ����ʵ��Ȼ���ӽ���close�׽��ֺ��׽��ֶ��ڸ�������˵��Ȼ�ǿɶ��Ϳ�д�ģ����ܸ�������Զ������д�����ݡ�

��ˣ���socket�Ķ�������û�з�������ˣ��������˾Ͳ����⵽EOF��ʶ����һֱ�ȴ��ӿͻ����������ݡ�
����ʱ������Ҳ�����⵽�������˷�����EOF��ʶ�������������˺Ϳͻ���������������deadlock����
�����shutdown����close�������������ķ�����

     if( fork() ) {  /* The child                    */
           while( gets(buffer)
              write(s,buffer,strlen(buffer));
           shutdown(s,1); /* Break the connection
                             *for writing, The server will detect EOF now. Note: reading from
                		  *the socket is still allowed. The server may send some more data
               		  *after receiving EOF, why not? */
           exit(0);
      }

