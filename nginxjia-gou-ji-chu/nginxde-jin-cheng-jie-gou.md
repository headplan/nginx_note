# Nginx的进程结构

Nginx有两种进程结构 , **单进程结构**和**多进程结构** .

**单进程结构**不适用于生产环境 , 只适用于开发和调试 . 生产环境中 , 必须保证nginx足够健壮 , 并且nginx可以利用多核的特性 , 单进程结构无法满足 . 所以nginx的默认配置中都是打开为多进程结构的nginx . 配置文件 :

```
worker_processes auto;
或者
worker_processes 8; # CPU核数的2倍
```

#### Nginx的多进程模型

![](/assets/nginxduojinchengmoxing.png)

**为什么Nginx采用多进程结构 , 而不是多线程结构呢 ? **

答案是 , 为了保证Nginx的高可用性和高可靠性 .

如果使用多线程结构的情况下 , 线程之间是共享同一个地址空间的 , 所以当某一个第三方模块引发了一个地址空间导致的段错误时 , 也就是在地址越界出现时 , 会导致整个Nginx的进程全部挂掉 . 当我们采用多进程的Nginx进程模型时 , 往往就不会出现类似的情况 .

Nginx在做进程设计时 , 同样遵循了实现高可用和高可靠的目的 . 在master进程中 , 第三方模块通常不会在其中加入自定义的功能代码的 , 虽然这是允许的 , 但通常没有第三方模块会这样做 . master进程的目的 , 就是用来管理worker进程的 . 也就是说 , 所有的worker进程是处理真正的请求的 , 而master进程负责监控每个worker进程是不是在正常的工作 , 需不需要做重新载入配置文件 , 需不需要做热部署 . 说道缓存的时候 , 缓存是需要在多个worker进程间共享的 , 而且缓存不光要被worker进程使用 , 还要被Cache Manager和Cache Loader进程使用 , Cache Manager和Cache Loader也是为反向代理时 , 后端发来的动态请求做缓存所使用的 . Cache Loader只是用来做缓存的载入 , Cache Manager用来做缓存的管理 . 实际上 , 每个请求处理时 ,  使用到缓存 , 还是通过worker进程来进行的 . 这些进程间的通讯是使用共享内存来解决的 .

**为什么有多个worker进程 ? **

Nginx采用了四列驱动的模型以后 , 希望每一个worker进程从头到尾都占有一颗CPU , 所以往往不只要把worker进程的数量配置成和CPU核数一致 , 还要把每个worker进程与某一颗CPU核绑定在一起 , 这样可以更好的使用没颗CPU核上面的CPU缓存来减少缓存失效的命中率 .

2核CPU , 开启2个进程 :

```
worker_processes     2;
worker_cpu_affinity 01 10;
```

2核cpu , 开启4个进程 :

```
worker_processes     4;
worker_cpu_affinity 01 10 01 10;
```

4个cpu , 开启4个进程 : 

```
worker_processes     4;
worker_cpu_affinity 0001 0010 0100 1000;
```

8核cpu , 开启8个进程 : 

```
worker_processes     8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
```



