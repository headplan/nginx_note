# 初识Nginx

Nginx是一个高性能的HTTP和反向代理服务器 , 主要特点是占用内存少 , 并发能力强 .

#### 基本原理

* 工作模型
* 进程解析

**工作模型**

Nginx的高性能主要是使用了epoll和kqueue网络I/O模型 .

* epoll - 使用于Linux内核2.6版本以及后的系统 , 在某些发行版中 , 如SuSE8.2 , 有让2.4版本的内核支持epoll的补丁
* kqueue - 使用于FreeBSD4.1+ , OpenBSD2.9+ , NetBSD2.0 , MacOS X

Apache使用了传统的select模型\(Apache2.4后也使用了epoll网络I/O模型\) .

Linxu下的高并发软件Squid , Memcached采用的也都是epoll网络I/O模型 .

---

#### 



