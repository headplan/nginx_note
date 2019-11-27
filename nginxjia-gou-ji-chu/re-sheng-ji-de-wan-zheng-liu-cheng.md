# 热升级的完整流程

将旧Nginx文件换成新Nginx文件\(注意备份\)

向master进程发送USR2信号

master进程修改pid文件名 , 加后缀.oldbin

master进程用新Nginx文件启动新master进程

向老master进程



