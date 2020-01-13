# Nginx的事件驱动模型

![](/assets/shijianxunhuan.png)

当Nginx刚刚启动时 , WAIT FOR EVENTS ON CONNECTIONS , 也就打开了80/443端口 , 这个时候在等待新的事件进来 . 比如 , 新的客户端连上了我们的Nginx , 向我们发起了连接 , WAIT FOR EVENTS ON CONNECTIONS就是在等这样的事件 . 这一步在epoll模型中对应着epoll wait这样一个方法 , 这个时候Nginx其实是处于SLEEP的进程状态的 .

当操作系统收到了一个建立TCP连接的握手报文 , 并且处理完握手流程以后 , 操作系统就会通知epoll wait这个阻塞方法 , 告诉其可以继续向下进行了 , 同时唤醒Nginx的worker进程 .

RECEIVE A QUEUE OF NEW EVENTS , 接收新的事件 , 向操作系统要事件 . 这里的KERNEL就是操作系统的内核 , 操作系统会把它准备好的事件 , 放在事件队列中 . 从事件队列中可以获取到我们要处理的事件 . 比如 , 建立连接 , 接收TCP的请求报文 , 都可以从事件队列中取出 , 然后处理事件 , PROCESS THE EVENTS QUEUE IN CYCLE .

右边的图 , 就是处理事件的循环 . THE EVENTS QUEUE PROCESSING CYCLE . 发现队列不为空 , 就把事件取出来 , 开始处理这个事件 , 在这个过程中 , 可能还会生成新的事件 , 比如发现一个连接是新建立的 , 可能要添加一个超时时间 , 默认60s , 在时间内浏览器不发送请求就关闭链接 , 

