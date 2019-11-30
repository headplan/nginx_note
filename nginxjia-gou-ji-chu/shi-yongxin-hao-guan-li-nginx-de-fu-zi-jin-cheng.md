# 使用信号管理Nginx的父子进程

Nginx是一个多进程的程序 , 多进程之间进行通讯可以使用共享内存 , 信号等 . 当我们做进程间的管理时 , 通常只使用信号 .

![](/assets/xinhaoguanli.png)

能够发送处理信号的有Master进程 , Worker进程 , 还有Nginx命令行 . 

#### Master进程

master进程会启动worker进程 , 所以他管理worker进程的方式首先是 , 监控worker进程有没有发送CHLD信号 . 因为Linux操作系统中规定 , 当子进程终止的时候 , 会向父进程发送CHLD信号 , 所以如果worker进程由于一些模块出现了BUG , 导致worker进程意外的终止掉 , 那么Master进程 , 可以立刻通过CHLD发现问题 , 然后重新把worker进程拉起 . 

 



