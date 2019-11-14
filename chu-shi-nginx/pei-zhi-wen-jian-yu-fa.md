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



