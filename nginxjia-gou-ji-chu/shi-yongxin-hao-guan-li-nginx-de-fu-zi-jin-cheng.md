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

#### Worker进程

worker进程也可以接收信号 :

* TERM , INT
* QUIT
* USR1
* WINCH

通常不直接对worker进程发送信号 , 因为我们希望通过master进程管理去管理worker进程 , 虽然发送这些信号一样可行 , 但往往我们还是对master进程进行管理 , master进程收到信号以后 , 会去再把信号发送给worker进程 .

#### Nginx命令行

当启动了nginx之后 , nginx会把PID记录到一个文件中 , 默认记录到nginx安装目录的logs文件夹下nginx.pid文件中 , 也就是master进程的PID . 我们在执行nginx命令时 , nginx的工具命令行首先读取PID文件获得master进程的PID , 然后像前面的操作一样 , 去向这个PID发送信号 , 对应的命令是 :

* reload : HUP
* reopen : USR1
* stop : TERM
* quit : QUIT



