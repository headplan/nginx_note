# Nginx的进程结构实例演示

前面提到 , nginx使用了多进程模型 , 也知道了nginx的父子进程之间是通过信号管理的 .

**那这些父子进程以及信号之间是如何工作的呢 ? **

![](/assets/jinchengid.png)

通过ps命令 , 可以看到nginx:master进程是用root用户起的 , 进程的id是3544 , 两个worker进程都是通过3544起来的 . 两个worker进程的id是18348 , 18349 .

现在执行 :

```
nginx -s reload
```

优雅的退出worker进程 , 再使用新的配置项启动新的子进程 . 这里虽然没有改变配置 , 但是可以看到 , 老的子进程会退出 , 然后生成新的子进程 .

![](/assets/xindezijincheng.png)

可以看到之前的子进程都不在了 , 由3544master进程新启动了两个子进程30552 , 30553 . 之前说过 , reload和HUP信号的作用是相同的 . 现在向nginx的master进程3544发送HUP信号 :

```
kill -SIGHUP 3544
```

![](/assets/fasonghupxinhao.png)

之前我们还提过 , 像quit , stop他们都有相应的信号 , 如果向一个worker进程 , 比如31249 , 发送一个退出信号 , 那么这个worker进程就会退出 , 而且进程在退出时 , 会自动的向其父进程3544 , 也就是master进程发送一个 , SIGCHILD信号 , 这个信号收到以后 , master进程就知道他的子进程退出了 , 他会新启一个worker进程 , 维持当前两个worker进程的进程结构 : 

```
kill -SIGTERM 31249
```

![](/assets/sigtermxinhao.png)

