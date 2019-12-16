# 使用信号管理Nginx的父子进程

Nginx是一个多进程的程序 , 多进程之间进行通讯可以使用共享内存 , 信号等 . 当我们做进程间的管理时 , 通常只使用信号 .

![](/assets/xinhaoguanli.png)

能够发送处理信号的有Master进程 , Worker进程 , 还有Nginx命令行 .

#### Master进程

master进程会启动worker进程 , 所以他管理worker进程的方式首先是 , 监控worker进程有没有发送CHLD信号 . 因为Linux操作系统中规定 , 当子进程终止的时候 , 会向父进程发送CHLD信号 , 所以如果worker进程由于一些模块出现了BUG , 导致worker进程意外的终止掉 , 那么Master进程 , 可以立刻通过CHLD发现问题 , 然后重新把worker进程拉起 . 

master进程还会通过接收一些信号来管理worker进程 : 

* TERM , INT - 表示立刻停止nginx进程
* QUIT - 优雅的停止nginx进程 , 优雅的可以理解为慢慢的停 . 保证不对用户发送立刻结束连接 , 例如TCP的reset复位请求这样的报文 . 
* HUP - 表示重载配置文件 . 
* USR1 - 表示重新打开配置文件 , 做日志文件的切割 . 

上面四个信号 , 可以通过nginx命令行像nginx发送 . 下面的两个信号 , 只能通过kill向master进程发送 , 也就是说需要先找到master的进程PID , 对这个PID发送信号 , 下面两个信号是针对热部署使用 . 

* USR2
* WINCH



