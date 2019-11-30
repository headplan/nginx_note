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

