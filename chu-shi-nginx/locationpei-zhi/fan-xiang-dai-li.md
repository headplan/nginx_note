# 反向代理

通常在公司的服务器里面如果配有集群 , 那么运维人员对服务器代理将不再陌生 , 在负载均衡的时候就一定会用到反向代理 . 或者正向代理 . 但是他们之间的区别是什么呢?

#### **正向代理**

> 正向代理类似一个跳板机 , 代理访问外部资源 .

[  
![](https://img.hacpai.com/file/2017/2/637420bcbc2548c48a9e0e13a98906b7.png)](https://img.hacpai.com/file/2017/2/637420bcbc2548c48a9e0e13a98906b7.png?imageView2/2/w/768/format/jpg/interlace/0/q)

1 . 用户 A 主动请求要访问原始服务器 B , 从图中可以看出 A 先访问代理服务器 z , 然后由 z 将请求发给服务器 B , 同时代理服务器 Z 也负责将返回的数据发送给用户 .

2 . 用户知道服务器 B , 也知道代理服务器 z , 但是他所做的请求都是由代理服务器来处理 .

3 . “缓存”--- 可以在代理服务器 z 做缓存 , 用户 a 不用直接访问服务器 b 就可以拿到所要的数据\(cache\)

[![](https://img.hacpai.com/file/2017/2/05b687ab09e642a393a3cba81d912319.png)](https://img.hacpai.com/file/2017/2/05b687ab09e642a393a3cba81d912319.png?imageView2/2/w/768/format/jpg/interlace/0/q)

4 . 由于用户 A 到服务器 B 可能需要经过很多路由 , 导致速度较慢 , 采用代理 , 可以“加速访问” .

5 . 由图可以看出用户 A 不能直接访问服务器 B , 需要代理服务器 z , 常见实例为“翻墙”

[![](https://img.hacpai.com/file/2017/2/d77afb44a17a4b01b48d3a928df6ad78.png)](https://img.hacpai.com/file/2017/2/d77afb44a17a4b01b48d3a928df6ad78.png?imageView2/2/w/768/format/jpg/interlace/0/q)

6 . 从图中可以看出 , 采用代理服务器可以做一些验证 , 比如上网权限 , 因为要连接互联网首先得经过代理服务器 . \(权限验证\)

#### **反向代理**

> 反向代理（Reverse Proxy）实际运行方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

#### [ ![](https://img.hacpai.com/file/2017/2/de54cd5216514286b07721ffa1ccccef.png)](https://img.hacpai.com/file/2017/2/de54cd5216514286b07721ffa1ccccef.png?imageView2/2/w/768/format/jpg/interlace/0/q)

1 . 反向代理正好与正向代理相反 , 对于客户端而言代理服务器就像是原始服务器 , 并且客户端不需要进行任何特别的设置 . 客户端向反向代理的命名空间 \(name-space\) 中的内容发送普通请求 , 接着反向代理将判断向何处 \(原始服务器\) 转交请求 , 并将获得的内容返回给客户端 .

2 . 用户 A 始终认为它访问的是原始服务器 B 而不是代理服务器 Z , 但实用际上反向代理服务器接受用户 A 的应答 , 从原始资源服务器 B 中取得用户 A 的需求资源 , 然后发送给用户 A . 由于防火墙的作用 , 只允许代理服务器 Z 访问原始资源服务器 B . 尽管在这个虚拟的环境下 , 防火墙和反向代理的共同作用保护了原始资源服务器 B , 但用户 A 并不知情 .

[![](https://img.hacpai.com/file/2017/2/d794433d7f344ef2b7f26acb3531bdf5.png)](https://img.hacpai.com/file/2017/2/d794433d7f344ef2b7f26acb3531bdf5.png?imageView2/2/w/768/format/jpg/interlace/0/q)

3 . 如图所示 , 就是负载均衡 , 当 http 请求过多时 , 反向代理服务器负责分发 http 请求 , 确保某台资源服务器\(可以是集群\)压力不会太大 , 而导致崩溃 .

[![](https://img.hacpai.com/file/2017/2/6bbb491d7e0a4e30b73da410cd5813a0.png)](https://img.hacpai.com/file/2017/2/6bbb491d7e0a4e30b73da410cd5813a0.png?imageView2/2/w/768/format/jpg/interlace/0/q)

4 . 当然反向代理服务器像正向代理服务器一样拥有 CACHE 的作用 , 它可以缓存原始资源服务器 B 的资源 , 而不是每次都要向原始资源服务器 B 请求数据 , 特别是一些静态的数据 , 比如图片和文件 , 如果这些反向代理服务器能够做到和用户 X 来自同一个网络 , 那么用户 X 访问反向代理服务器 X , 就会得到很高质量的速度 . 这正是 CDN 技术的核心 .

