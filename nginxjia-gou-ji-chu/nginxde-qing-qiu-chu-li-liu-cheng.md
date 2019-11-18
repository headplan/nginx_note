# Nginx的请求处理流程

![](/assets/nginxqingqiuchuliliucheng.png)

Web , Email , TCP大致有三种流量进入 . Nginx中有三个大的状态机 :

* 传输层状态机 : 处理TCP , UDP的四层的传输层状态机 . 
* HTTP状态机 : 处理应用层的HTTP状态机 . 
* Mail状态机 : 处理邮件的Mail状态机 . 

**为什么叫状态机呢 ? **

因为Nginx中核心的处理引擎是非阻塞的事件驱动处理引擎 , 也就是我们熟知的epoll . 一旦使用这种异步处理引擎 , 通常都需要状态机把请求正确的识别和处理的 . 基于这样的处理方式 , 在解析出请求 , 需要访问静态资源的时候 , 就会走左下方的**静态资源** . 如果做反向代理的时候 , 对方向代理的内容可以做**磁盘缓存** . 

但是在处理静态资源时候会有一个问题 , 当整个内存已经不足以完全的缓存住所有的文件缓存信息时 , 像sendfile这样的调用 , 或者AIO会退化成阻塞的磁盘调用 . 因为操作系统要缓存inode , 当内存不足时 , 连inode都只能从磁盘读出来时 , 此时的异步IO都会退化成阻塞调用 . 可以看下实验过程 : https://www.nginx.com/blog/thread-pools-boost-performance-9x/

所以在上图中会看到一个线程池来处理磁盘阻塞调用 . 

对于每一个处理完成的请求 , 会记录Access访问日志和Error错误日志 . 这里是记录在磁盘中的 , 当然也可以使用syslog协议 , 记录到远程的服务器上 . 



扩展阅读 : 

[http://www.aosabook.org/en/nginx.html](http://www.aosabook.org/en/nginx.html)

