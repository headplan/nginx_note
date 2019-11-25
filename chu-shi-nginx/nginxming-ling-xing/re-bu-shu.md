# 热部署

热部署的意思是Nginx是正在运行的 ,

```
ps -ef | grep nginx
```

需要更新Nginx最新版本 . 首选需要把现有的Nginx的二进制文件进行备份 , 因为更新所更换的只是二进制文件 .

```
cp nginx nginx.old
```

然后编译新版本的nginx二进制文件 , 并将文件复制替换

```
cp nginx /usr/local/nginx/sbin/ -f
```

> 强制复制 -f 参数

给nginx的master进程发送信号

```
kill -USR2 masterPid
```

现在会新启动一个nginx的master进程 , 使用前面复制过来的nginx二进制文件 . 老的worker还在运行 , 新的master会生成新的worker , 平滑的把请求过渡到新的nginx二进制文件启动的nginx进程中 .

![](/assets/fasongxinhao1.png)

现在 , 新老进程都在运行 , 但是老的woker进程已经不再监听80/443这样的web端口了 . 新的请求 , 新的连接只会进入新的进程中 . 现在要发送一个信号给老的进程 , 让其优雅的关闭其worker进程 .

```
kill -WINCH masterPid
```

> 通过`lsof -p masterPid`还会查看到端口在监听 , 但是是master进程 . 这里的不再监听是指woker进程不会去处理socket了 , 因为没有把它加到epoll中 . master进程打开监听端口 , 但不处理 , 由worker进程处理 . 另外 , 旧的master是新的master的父进程 , 所以新master才能共享打开的监听端口 .

![](/assets/fasongxinhao2.png)

现在可以看到 , 老的master进程已经没有worker进程了 , 说明现在所有请求都已经切换到新的nginx中了 . 如果 , 现在发现一些问题 , 需要把新版本回退 , 可以给老版本的master进程发送reload命令 , 从新吧worker进程拉起来 , 再把新版本关掉 .

> 在执行完`kill -USR2 masterPid`后 , 可以用`lsof -p masterPid`查看进程打开的句柄 , 也包括监听的端口 . 用netstat命令也可以 , 或者直接在/proc目录中找进程的相关信息也可以 .
>
> `kill -QUIT masterPid` 可以杀掉老的master .
>
> 上面提到的新版本回退 , 使用`kill -HUP masterPid`来执行reload , 把老的worker进程拉起来 .



