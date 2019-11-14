# Nginx命令行

```
选项:
  -?,-h         :帮助
  -v            :显示版本信息
  -V            :显示版本信息和编译配置信息
  -t            :测试配置文件是否有语法错误
  -T            :测试配置文件是否有语法错误并打印出来
  -q            :在检测配置文件期间屏蔽非错误信息
  -s signal     :给一个nginx主进程发送信号:
    stop:立刻停止服务
    quit:优雅的停止服务
    reopen:重新开始记录日志文件
    reload:重载配置文件
  -p prefix     :设置前缀路径,就是安装的路径(default:/Users/headplan/Desktop/mynginx/)
  -c filename   :设置配置文件(default:conf/nginx.conf)
  -g directives :设置配置文件外的全局指令
```



