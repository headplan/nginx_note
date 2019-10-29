# 负载均衡配置

```
upstream test.com {
    server 192.168.1.20:80 weight=2;
    server 192.168.1.21:80 weight=1;
}
```

upstream模块通过简单的调度算法实现客户端到服务器的负载均衡 . 上面的例子中 , test.com是这个负载均衡的名字 , 可以在后面的配置中调用 .

Nginx支持一下4种负载均衡算法 :

* 加权轮询\(默认的算法\) - 请求按时间分别分配到不同的服务器上 . 
* ip\_hash - 使用请求的ip算出hash值 , 根据hash值分配到不同的服务器上 , 固定ip的请求 , 会分配到固定的服务器 . 这种策略有效的解决了网站服务的session共享问题 . 
* fair - 按后端服务器的响应时间来分配请求 , 响应时间短的优先分配 . Nginx默认是不支持这种负载均衡算法的 , 需要安装Nginx模块和upstream\_fair模块 . 
* url\_hash - 使用请求的URL算出hash值 , 根据hash值分配到不同的服务器上 , 固定的URL请求 , 会分配到固定的服务器上 . 这种策略有利于提高后端服务器的缓存命中率 . Nginx默认是不支持这种负载均衡算法的 , 需要安装Nginx的hash软件包 . 

upstream模块可以为所配置的服务器指定状态值 , 常见的有 :

* down - 服务器不参与到负载均衡中 , 当后台人员进行故障排查时这个状态非常有用 . 
* weight - 制定轮询的权重 , 权重越大分配到的几率越多 . 前面例子中分配的请求比例就是2:1 . 
* backup - 备份机器 . 当其他的服务器不可用时 , 才把请求分配到这台服务器上 . 
* max\_fails - 允许请求失败的次数 , 默认值是1 . 
  * 这些错误在proxy\_next\_upstream或fastcgi\_next\_upstream\(404错误不会使max\_fails增加\)中定义
* fail\_timeout - 经历了前面max\_fails次失败后 , 暂停服务的时间
  * 可以使用proxy\_connect\_timeout和proxy\_read\_timeout来控制

> 注意 : 当负载均衡是ip\_hash算法时 , 服务器的状态值不能是backup和weight .
>
> **关于max\_fails 参数的理解**
>
> 根 据上面的解释 , max\_fails默认为1 , fail\_timeout默 认为10秒 . 也就是说 , 默认情况下后端服务器在10秒钟之内可以容许有一次的失败 , 如果超过1次则视为该服务器有问题 , 将该服务器标记为不可用 . 等待10秒后再将请求发给该服务器 , 以此类推进行后端服务器的健康检查 . 但如果我将max\_fails设置为0 , 则代表不对后端服务器进行健康检查 , 这样一来fail\_timeout参数也就没什么意义了 . 那若后端服务器真的出现问题怎么办呢 ? 上文也说了 , 可以借助proxy\_connect\_timeout和proxy\_read\_timeout进行控制

---

**负载均衡样例**

```
upstream mysvr { 
   server 192.168.10.121:3333;
   server 192.168.10.122:3333;
}
server {
        ....
   location  ~*^.+$ {         
      proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表         
   }
}
==========
upstream mysvr { 
   server  http://192.168.10.121:3333;
   server  http://192.168.10.122:3333;
}
server {
   ....
   location  ~*^.+$ {         
      proxy_pass  mysvr;  #请求转向mysvr 定义的服务器列表         
   }
}

# 热备
upstream mysvr { 
   server 127.0.0.1:7878; 
   server 192.168.10.121:3333 backup;  #热备     
}

# 轮询
upstream mysvr { 
   server 127.0.0.1:7878;
   server 192.168.10.121:3333;       
}

# 加权轮询
upstream mysvr { 
   server 127.0.0.1:7878 weight=1;
   server 192.168.10.121:3333 weight=2;
}

# ip_hash:nginx会让相同的客户端ip请求相同的服务器
upstream mysvr { 
   server 127.0.0.1:7878; 
   server 192.168.10.121:3333;
   ip_hash;
}
```



