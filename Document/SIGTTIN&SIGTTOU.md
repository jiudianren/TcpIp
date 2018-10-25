《APUE 》 10.2节

Unix环境下，当一个进程以后台形式启动，但尝试去读写控制台终端时，将会触发SIGTTIN（读）和SIGTTOU（写）信号量，接着，进程将会暂停（linux默认情况下），read/write将会返回错误。这个时候，shell将会发送通知给用户，提醒用户切换此进程为前台进程，以便继续执行。由后台切换至前台的方式是fg命令，前台转为后台则为CTRL+Z快捷键。
??那么问题来了，如何才能在不把进程切换至前台的情况下，读写控制器不会被暂停？答案：只要忽略SIGTTIN和SIGTTOU信号量即可：signal(SIGTTOU, SIG_IGN)。
??stty stop/-stop命令是用于设置收到SIGTTOU信号量后是否执行暂停，因为有些系统的默认行为不一致，比如mac是默认忽略，而linux是默认启用。stty -a可以查看当前tty的配置参数


http://vinllen.com/hou-tai-jin-cheng-du-xie-kong-zhi-tai-hong-fa-sigttin-sigttouxin-hao-liang/