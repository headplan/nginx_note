# Nginx反向代理

**反向代理样例**

```
upstream backend {
     server 10.46.194.17:8088 weight=5;
     server 10.46.192.41:8080 weight=5;
}

# proxy_cache_path /tmp/cache levels=1:2 keys_zone=cache:60m max_size=1G;

server {
     listen 8079;
     server_name www.test.com;
     underscores_in_headers on;
     ignore_invalid_headers off;

     location / {
          # include ...proxy.conf
          proxy_connect_timeout 300s;
          proxy_send_timeout 900;
          proxy_read_timeout 900;
          proxy_buffer_size 32k;
          proxy_buffers 4 64k;
          proxy_busy_buffers_size 128k;
          proxy_redirect off;
          proxy_hide_header Vary;
          proxy_set_header Accept-Encoding '';
          proxy_set_header Referer $http_referer;
          proxy_set_header Cookie $http_cookie;
          proxy_set_header Host $host:$server_port;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_pass http://backend;

          # proxy_buffering    off;
          # proxy_buffer_size  128k;
          # proxy_buffers 100  128k;
          location ~* \.(html|css|jpg|gif|ico|js)$ {
               proxy_cache cache;
               proxy_cache_key $host$uri$is_args$args;
               proxy_cache_valid 200 301 302 30m;
               expires 30m;
               proxy_pass http://backend;
          }
}
```

**这里记录前三个参数配置的意义**

```
proxy_connect_timeout 300s; # 后端服务器连接的超时时间_发起握手等候响应超时时间
proxy_send_timeout 900; # 后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
proxy_read_timeout 900; # 连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
# proxy_read_timeout要根据自身程序而定,不要过大,也不要太小.如果是php程序,请参照php.ini中的max_execution_time选项值.
```

**这里记录后三个参数的作用 - 缓冲控制**

```
proxy_buffer_size 32k;
proxy_buffers 4 64k;
proxy_busy_buffers_size 128k;
```

> **nginx proxy\_buffer\_size 解决后端服务传输数据过多 , 其实是header过大的问题 . **

---

这里也可以禁用proxy的缓冲区 , 那么当Nginx一收到后端的反馈就同时传给客户端 , 但是因为禁用了 , 所以不会从被代理的服务器读取整个反馈信息 , 读取的信息大小由proxy\_buffer\_size控制 .

```
proxy_buffering    off;
proxy_buffer_size  128k;
proxy_buffers 100  128k;
```

如果上面将所有请求都转发给后端应用 . 为避免静态请求给后端应用带来的过大负载 , 我们可以将nginx配置为缓存那些不变的响应数据 .

```
http {
     #
     # The path we'll cache to.
     #
     proxy_cache_path /tmp/cache levels=1:2 keys_zone=cache:60m max_size=1G;
}
            ## send all traffic to the back-end
            location / {
                 proxy_pass  http://backend;
                 proxy_redirect off;
                 proxy_set_header        X-Forwarded-For $remote_addr;
                 location ~* \.(html|css|jpg|gif|ico|js)$ {
                        proxy_cache          cache;
                        proxy_cache_key      $host$uri$is_args$args;
                        proxy_cache_valid    200 301 302 30m;
                        expires              30m;
                        proxy_pass  http://backend;
                 }
            }
```

这里 , 我们将请求缓存到 `/tmp/cache`并定义了其大小限制为1G . 同时只允许缓存有效的返回数据 , 例如

```
proxy_cache_valid  200 301 302 30m;
```

所有响应信息的返回代码不是 "`HTTP (200|301|302) OK`" 的都不会被缓存 .

---

**proxy\_redirect**

书上的描述是 , 当上有服务器返回的相应是重定向或刷新请求时 , **proxy\_redirect**可以重设HTTP头部的location和refresh字段 .

这里的off的意思 , 就是维持location和refresh字段不变 .

**proxy\_hide\_header**

Nginx会将上游服务器的响应转发给客户端 , 但默认不会转发一下HTTP头部字段 : Date,Server,X-Pad,X-Accel-\* .

使用**proxy\_hide\_header**后可以任意的指定哪些HTTP头字段不能被转发 . 与之对应的是**proxy\_pass\_header** , 将原来禁止转发的设置为允许 .

**proxy\_set\_header**

重新定义或者添加发往后端服务器的请求头 .

```
proxy_set_header Accept-Encoding ''; # 清除编码.如果某个请求头的值为空,那么这个请求头将不会传送给后端服务器
proxy_set_header Referer $http_referer;
proxy_set_header Cookie $http_cookie;
proxy_set_header Host $host:$server_port; # 传递host,如果upstream是个域名也可以写成域名.服务器名可以和后端服务器的端口一起传送.
# 下面这俩是用来获取用户真实IP的
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

**这里记录两个自定义header头的参数**

```
underscores_in_headers on; # 开启配置这个后,HTTP头部支持有下划线的
ignore_invalid_headers off; 
# 是否忽略不合法的http首部.默认为on,off意味着请求首部中出现不合规的首部将拒绝响应.只能用于server和http中,建议改为off.
```

1 . nginx是支持读取非nginx标准的用户自定义header的 , 但是需要在http或者server下开启header的下划线支持 :

* underscores\_in\_headers on;

2 . 比如我们自定义header为X-Real-IP,通过第二个nginx获取该header时需要这样

* $http\_x\_real\_ip; \(一律采用小写，而且前面多了个http\_\)

3 . 如果需要把自定义header传递到下一个nginx

* 如果是在nginx中自定义采用proxy\_set\_header X\_CUSTOM\_HEADER $http\_host;
* 如果是在用户请求时自定义的header，例如curl –head -H “X\_CUSTOM\_HEADER: foo” [http://domain.com/api/test](http://domain.com/api/test)
  则需要通过`proxy_pass_header X_CUSTOM_HEADER`来传递

**其他的相关配置**

```
proxy_http_version 1.0 ; # Nginx服务器提供代理服务的http协议版本1.0，1.1,默认设置为1.0版本
proxy_method get;        # 支持客户端的请求方法,post/get
proxy_ignore_client_abort on;  # 客户端断网时,nginx服务器是否终断对被代理服务器的请求.默认为off.
proxy_ignore_headers "Expires" "Set-Cookie";  # Nginx服务器不处理设置的http相应投中的头域,这里空格隔开可以设置多个.
proxy_intercept_errors on;    # 如果被代理服务器返回的状态码为400或者大于400,设置的error_page配置起作用.默认为off.
proxy_headers_hash_max_size 1024; # 存放http报文头的哈希表容量上限,默认为512个字符
proxy_headers_hash_bucket_size 128; # nginx服务器申请存放http报文头的哈希表容量大小.默认为64个字符.
proxy_next_upstream timeout;  # 反向代理upstream中设置的服务器组,出现故障时,被代理服务器返回的状态值.
# error|timeout|invalid_header|http_500|http_502|http_503|http_504|http_404|off
# 还有一点要注意的是,这个配置可能会让写请求就会发生多次提交事件
proxy_ssl_session_reuse on; # 默认为on,如果我们在错误日志中发现“SSL3_GET_FINSHED:digest check failed”的情况时
# 可以将该指令设置为off
```



