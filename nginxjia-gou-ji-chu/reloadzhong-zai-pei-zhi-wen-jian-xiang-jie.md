# reload重载配置文件详解

在更改了Nginx配置文件后 , 都会执行`nginx -s reload`命令 . 目的是 , 希望在nginx不停止服务 , 还在不停的处理新的请求 , 同时把nginx的配置文件平滑的从就的nginx.conf更新问新的nginx.conf .

有时候会发现 , 在执行了nginx -s reload之后 , worker进程的数量变多了 . 这是因为老的配置运行的woker进程长时间没有退出 , 当使用stream作为4层反向代理时 , 这样的场景会更多 .

##### 那么nginx优雅的退出和立即退出到底有什么差别呢 ?

#### reload流程

* 向master进程发送HUP信号 , 相当于执行`nginx -s reload`命令
* master进程校验配置语法是否正确 , 也就是说并不一定非要在nginx -s reload之前执行nginx -t去检验语法
* master进程打开新的监听端口 , 可能在conf中引入了新的监听端口 , 所有的worker进程是master进程的子进程 , 子进程会继承父进程所有已经打开的端口 , 这是Linux操作系统定义的 .
* master进程用新配置启动新的worker子进程
* master进程向老worker子进程发送QUIT信号
* 老worker进程关闭监听句柄 , 处理完当前连接后结束进程

#### 不停机载入新配置

![](/assets/butingjizaoruxinpeizhi.png)

如图所示 , Master进程上有4个worker子进程 , 当更新了nginx.conf配置文件后 , 向Master发送SIGHUP信号或者执行nginx -s reload命令 . Master进程会使用新的配置文件启动新的4个worker子进程 . 此时新老worker子进程是并存的 , 也就是图上的黄色和绿色的worker子进程 . 老配置的worker进程会在完成已存在连接时优雅的退出 , 哪怕是keepalive请求 .

异常情况下 , worker进程可能会请求错误 , 客户端长时间没有处理 , 就会导致这个请求长时间的占用在这个worker进程上面 ,  这个worker进程就会一直存在 , 不过新的连接已经跑在新的黄色的worker子进程中 , 影响并不会很大 . 唯一会导致这个worker子进程一直存在 , 但也只影响已经存在的连接 , 不会影响新的连接 .

nginx比较新的版本中 , 已经提供了配置 , worker\_shutdown\_timeout . 这个配置在master进程启动黄色worker子进程的时候 , 会开启一个定时器 , 设置多少秒后关闭绿色的连接 , 时间到了之后 , 会立刻强制的退出老的worker子进程 .

