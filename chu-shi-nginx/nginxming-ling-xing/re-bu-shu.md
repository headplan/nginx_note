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



