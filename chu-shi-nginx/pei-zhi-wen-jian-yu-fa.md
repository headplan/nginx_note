# 配置文件语法

* Nginx配置文件是一个asiic文本文件 , 主要有两部分指令和指令块组成 . 
* 每条指令以`;`分号结尾 , 指令与参数间以空格符号分隔 . 
* 指令块以`{}`大括号将多条指令组织在一起 . 
  * 指令块是否有名字 , 是由相应的Nginx模块决定的 . 
* include语句允许组合多个配置文件以提升可维护性 . 
* 使用\#符号添加注释 , 提高可读性 . 
* 使用$符号使用变量 . 
* 部分指令的参数支持正则表达式 . 

#### 配置参数时间单位

* ms - milliseconds 毫秒
* s - seconds 秒
* m - minutes 分钟
* h - hours 小时
* d - days 天
* w - weeks 周
* M - months , 30 days
* y - years , 365 days

#### 配置参数空间单位

* 默认 字节\(bytes\)
* k/K 千字节\(kilobytes\)
* m/M 兆字节\(megabytes\)
* g/G 千兆字节\(gigabytes\)

#### HTTP配置的指令块

有四个指令快 , http , server , upstream , location . 

* 其中http {}下的所有指令 , 都是由http模块去解析执行的 . 
* 其中upstream表示上有服务 , 当nginx需要与tomcat , djange等企业内网的其他服务交互的时候定义 . 
* server对应的一个域名或一组域名
* location是一个url表达式 . 



