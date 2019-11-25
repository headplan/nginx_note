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
kill -USR2 13195
```

现在会新启动一个nginx的master进程 , 使用前面复制过来的nginx二进制文件 . 老的worker还在运行 , 新的master会生成新的worker , 平滑的把请求过渡到新的nginx二进制文件启动的nginx进程中 . 

![](/assets/fasongxinhao1.png)

现在 , 新老进程都在运行 , 但是老的woker进程已经不再监听80/443这样的web端口了 . 新的请求 , 新的连接只会进入新的进程中 . 现在要发送一个信号给老的进程 , 让其优雅的关闭其worker进程 . 

```
kill -WINCH 13195
```



