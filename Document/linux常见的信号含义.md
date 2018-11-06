
SIGHUP     终止进程 终端线路挂断

	SIGINT 终止进程 中断进程 
	程序终止(interrupt)信号, 在用户键入INTR字符(通常是Ctrl-C)时发出，用于通知前台进程组终止进程。

SIGQUIT 建立CORE文件终止进程,并且生成core文件
SIGILL 建立CORE文件 非法指令
SIGTRAP 建立CORE文件 跟踪自陷
SIGBUS建立CORE文件 总线错误
SIGSEGV 建立CORE文件 段非法错误
SIGFPE 建立CORE文件 浮点异常
SIGIOT 建立CORE文件 执行I/O自陷
	
	SIGKILL  终止进程 杀死进程
	SIGPIPE  终止进程 向一个没有读进程的管道写数据
	SIGALARM 终止进程 计时器到时
	SIGTERM  终止进程 软件终止信号
	
SIGSTOP 停止进程 非终端来的停止信号
SIGTSTP 停止进程 终端来的停止信号
SIGCONT 忽略信号 继续执行一个停止的进程
SIGURG 忽略信号 I/O紧急信号
SIGIO 忽略信号 描述符上可以进行I/O

	SIGCHLD  忽略信号 当子进程停止或退出时通知父进程
SIGTTOU 停止进程 后台进程写终端
SIGTTIN 停止进程 后台进程读终端
SIGXGPU 终止进程 CPU时限超时
SIGXFSZ 终止进程 文件长度过长
SIGWINCH 忽略信号 窗口大小发生变化
SIGPROF 终止进程 统计分布图用计时器到时
SIGUSR1 终止进程 用户定义信号1
SIGUSR2 终止进程 用户定义信号2
SIGVTALRM 终止进程 虚拟计时器到时



###有两个信号可以停止进程:SIGTERM和SIGKILL。

SIGTERM比较友好,进程能捕捉这个信号,根据您的需要来关闭程序。在关闭程序之前,您可以结束打开的记录文件和完成正在做的任务。在某些情况下,假 如进程正在进行作业而且不能中断,那么进程可以忽略这个SIGTERM信号。
对于SIGKILL信号,进程是不能忽略的。这是一个 “我不管您在做什么,立刻停止”的信号。假如您发送SIGKILL信号给进程,Linux就将进程停止在那里。

以上是linux 常见的信号含义的内容，更多 含义 信号 常见 Linux 的内容，请您使用右上方搜索功能获取相关信息。



1) SIGHUP

本信号在用户终端连接(正常或非正常)结束时发出, 通常是在终端的控制进程结束时, 通知同一session内的各个作业, 这时它们与控制终端不再关联。

登录Linux时，系统会分配给登录用户一个终端(Session)。在这个终端运行的所有程序，包括前台进程组和后台进程组，一般都属于这个Session。当用户退出Linux登录时，前台进程组和后台有对终端输出的进程将会收到SIGHUP信号。这个信号的默认操作为终止进程，因此前台进程组和后台有终端输出的进程就会中止。不过可以捕获这个信号，比如wget能捕获SIGHUP信号，并忽略它，这样就算退出了Linux登录，wget也能继续下载。

此外，对于与终端脱离关系的守护进程，这个信号用于通知它重新读取配置文件。

2) SIGINT

程序终止(interrupt)信号, 在用户键入INTR字符(通常是Ctrl-C)时发出，用于通知前台进程组终止进程。

3) SIGQUIT

和SIGINT类似, 但由QUIT字符(通常是Ctrl-\)来控制. 进程在因收到SIGQUIT退出时会产生core文件, 在这个意义上类似于一个程序错误信号。

4) SIGILL

执行了非法指令. 通常是因为可执行文件本身出现错误, 或者试图执行数据段. 堆栈溢出时也有可能产生这个信号。

5) SIGTRAP

由断点指令或其它trap指令产生. 由debugger使用。

6) SIGABRT

调用abort函数生成的信号。

7) SIGBUS

非法地址, 包括内存地址对齐(alignment)出错。比如访问一个四个字长的整数, 但其地址不是4的倍数。它与SIGSEGV的区别在于后者是由于对合法存储地址的非法访问触发的(如访问不属于自己存储空间或只读存储空间)。

8) SIGFPE

在发生致命的算术运算错误时发出. 不仅包括浮点运算错误, 还包括溢出及除数为0等其它所有的算术的错误。

9) SIGKILL

用来立即结束程序的运行. 本信号不能被阻塞、处理和忽略。如果管理员发现某个进程终止不了，可尝试发送这个信号。

10) SIGUSR1

留给用户使用

11) SIGSEGV

试图访问未分配给自己的内存, 或试图往没有写权限的内存地址写数据.

12) SIGUSR2

留给用户使用

13) SIGPIPE

管道破裂。这个信号通常在进程间通信产生，比如采用FIFO(管道)通信的两个进程，读管道没打开或者意外终止就往管道写，写进程会收到SIGPIPE信号。此外用Socket通信的两个进程，写进程在写Socket的时候，读进程已经终止。

14) SIGALRM

时钟定时信号, 计算的是实际的时间或时钟时间. alarm函数使用该信号.

15) SIGTERM

程序结束(terminate)信号, 与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出，shell命令kill缺省产生这个信号。如果进程终止不了，我们才会尝试SIGKILL。

17) SIGCHLD

子进程结束时, 父进程会收到这个信号。

如果父进程没有处理这个信号，也没有等待(wait)子进程，子进程虽然终止，但是还会在内核进程表中占有表项，这时的子进程称为僵尸进程。这种情况我们应该避免(父进程或者忽略SIGCHILD信号，或者捕捉它，或者wait它派生的子进程，或者父进程先终止，这时子进程的终止自动由init进程来接管)。

18) SIGCONT

让一个停止(stopped)的进程继续执行. 本信号不能被阻塞. 可以用一个handler来让程序在由stopped状态变为继续执行时完成特定的工作. 例如, 重新显示提示符

19) SIGSTOP

停止(stopped)进程的执行. 注意它和terminate以及interrupt的区别:该进程还未结束, 只是暂停执行. 本信号不能被阻塞, 处理或忽略.

20) SIGTSTP

停止进程的运行, 但该信号可以被处理和忽略. 用户键入SUSP字符时(通常是Ctrl-Z)发出这个信号

21) SIGTTIN

当后台作业要从用户终端读数据时, 该作业中的所有进程会收到SIGTTIN信号. 缺省时这些进程会停止执行.

22) SIGTTOU

类似于SIGTTIN, 但在写终端(或修改终端模式)时收到.

23) SIGURG

有"紧急"数据或out-of-band数据到达socket时产生.

24) SIGXCPU

超过CPU时间资源限制. 这个限制可以由getrlimit/setrlimit来读取/改变。

25) SIGXFSZ

当进程企图扩大文件以至于超过文件大小资源限制。

26) SIGVTALRM

虚拟时钟信号. 类似于SIGALRM, 但是计算的是该进程占用的CPU时间.

27) SIGPROF

类似于SIGALRM/SIGVTALRM, 但包括该进程用的CPU时间以及系统调用的时间.

28) SIGWINCH

窗口大小改变时发出.

29) SIGIO

文件描述符准备就绪, 可以开始进行输入/输出操作.

30) SIGPWR

Power failure

31) SIGSYS

非法的系统调用。

在以上列出的信号中，程序不可捕获、阻塞或忽略的信号有：SIGKILL,SIGSTOP

不能恢复至默认动作的信号有：SIGILL,SIGTRAP

默认会导致进程流产的信号有：SIGABRT,SIGBUS,SIGFPE,SIGILL,SIGIOT,SIGQUIT,SIGSEGV,SIGTRAP,SIGXCPU,SIGXFSZ

默认会导致进程退出的信号有：SIGALRM,SIGHUP,SIGINT,SIGKILL,SIGPIPE,SIGPOLL,SIGPROF,SIGSYS,SIGTERM,SIGUSR1,SIGUSR2,SIGVTALRM

默认会导致进程停止的信号有：SIGSTOP,SIGTSTP,SIGTTIN,SIGTTOU

默认进程忽略的信号有：SIGCHLD,SIGPWR,SIGURG,SIGWINCH

此外，SIGIO在SVR4是退出，在4.3BSD中是忽略；SIGCONT在进程挂起时是继续，否则是忽略，不能被阻塞
