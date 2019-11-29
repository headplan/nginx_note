# 热升级的完整流程

Nginx热升级 , 保证Nginx在不停止服务的情况下 , 更换binary文件 . 在升级过程中 , 可能会遇到一些问题 , 需要注意 , 比如 :

老的worker进程一直退不掉 , 或者新的worker进程升级以后出现了问题需要回滚 , 或者升级了新的Nginx文件以后会发现很多预期的功能或者指向的配置文件发生错误等 .

#### 热升级流程

**将旧Nginx文件换成新Nginx文件\(注意备份\)**

* 这里指binary文件 , 新编译的Nginx文件所指定的配置选项 , 比如目录 , Log的目录等等必须保持和旧的一致 , 替换注意备份 , 还需注意的是 , 覆盖文件时 , 需要强制 , `cp -f`

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

**向master进程发送USR2信号**

给nginx的master进程发送信号

```
kill -USR2 masterPid
```

现在会新启动一个nginx的master进程 , 使用前面复制过来的nginx二进制文件 . 老的worker还在运行 , 新的master会生成新的worker , 平滑的把请求过渡到新的nginx二进制文件启动的nginx进程中 .

![](/assets/fasongxinhao1.png)

**master进程修改pid文件名 , 加后缀.oldbin**

虽然master进程和worker进程他们都可以接收信号 , 但是为了管理方便 , 通常不对worker进程直接发送信号 , 所以依赖master进程的pid , 所以老的pid文件会改名为.oldbin . 以上是在执行了上面的命令之后 , 自动生成的 , 在/var/run目录下 : 

![](/assets/nginxpidoldbin.png)

master进程用新Nginx文件启动新master进程

向老master进程发送QUIT信号 , 关闭老master进程

回滚 : 向老master发送HUP , 向新master发送QUIT

