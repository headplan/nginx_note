# Nginx网络事件实例演示

前面说了很多关于网络报文的发送 , 以及这些报文对应着的Nginx中的网络事件 . 比如 , 像Accept建立一条新连接 , 其实是收到了一条读事件 . 下面通过抓包分析一下 , 建立三次握手时 , 是怎样让Nginx收到读事件的 .

首先创建一个静态资源Web服务器 : [http://116.101.99.33:8080/](http://116.196.81.55:8080/) , 这个IP对应的就是Nginx服务器 . 访问这个服务器时 , 进行抓包 :

![](/assets/wireshark.png)

对Nginx服务器的IP以及8080端口进行抓包 .

```
192.168.0.101    116.101.99.33    TCP    78    56348 → 8080 [SYN] Seq=0 Win=65535 Len=0 MSS=1460 WS=32 TSval=284594423 TSecr=0 SACK_PERM=1
```

上面的IP地址是现在所在机器的IP和Nginx服务器的IP地址 . 对它建立三次握手时 , 首先发送SYN包 . 查看一下报文的TCP层 :

```
Transmission Control Protocol, Src Port: 56348, Dst Port: 8080, Seq: 0, Len: 0
```



