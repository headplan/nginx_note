# 负载均衡策略:round-robin

Nginx负责与上游服务交互的模块统称为upstream模块 , 不管是stream , upstream还是http\_upstream .

在upstream模块中 , 除了指定上游服务器以外, 还提供了一个最基本的负载均衡算法 , round-robin算法 .

#### 指定上游服务地址

通过upstream与server指令 .

`upstream name {...}` : 只能出现在http上下文中 , 定义name给反向代理模块使用 . 大括号中的内容就是下面配置的server的内容 , 指定name是一个应用集群 , 包含多个server .

`server address [parameters];`  : 每一个server就是一台服务器 , 每一台服务器中会跟着一个parameter , 去控制负载均衡的行为 .

* server address : 指定一组上游服务器地址 , 其中 , 地址可以是域名 , IP地址或者unix socket地址 . 可以在域名或者IP地址后加端口 , 默认使用80端口 . 
* 通用参数parameter : 
  * backup : 指定当前server为备份服务 , 仅当非备份server不可用时 , 请求才会转发到该server
  * down : 标识某台服务已经下线 , 不在服务

#### 加权Round-Robin负载均衡算法

**功能** : 在加权轮询的方式访问server指令指定的上游服务 .继承在Nginx的upstream框架中 . 其是其他算法的一个基础 , 比如哈希算法或一次性哈希算法 , 在某些情况下都会退化成加权Round-Robin负载均衡算法 .

这里**Round-Robin**的意思就是一次轮询 , 挨个进行的方式 . **加权** , 就是通过一个权重 , 来标识某台服务不同于其他服务 , 权重更大 , 将更多的请求发送给这台服务 . 

**Round-Robin提供了四个**`parameters`

weight

max\_conns

max\_fails

fail\_timeout



