# 热升级的完整流程

Nginx热升级 , 保证Nginx在不停止服务的情况下 , 更换binary文件 . 在升级过程中 , 可能会遇到一些问题 , 需要注意 , 比如 : 

老的worker进程一直退不掉 , 或者新的worker进程升级以后出现了问题需要回滚 , 或者升级了新的Nginx文件以后会发现很多预期的功能或者指向的配置文件发生错误等 . 

#### 热升级流程

将旧Nginx文件换成新Nginx文件\(注意备份\)

向master进程发送USR2信号

master进程修改pid文件名 , 加后缀.oldbin

master进程用新Nginx文件启动新master进程

向老master进程发送QUIT信号 , 关闭老master进程

回滚 : 向老master发送HUP , 向新master发送QUIT

