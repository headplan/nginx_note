# server虚拟主机配置

```
server {
    listen 80;
    server_name localhost test.com;
    index index.html
    root /var/www/test;
}
```

配置的含义 :

* listen - 指定虚拟主机监听的端口
* server\_name - 指定虚拟主机对应的域名 , 多个域名之间以空格分割
* index - 默认的首页支持
* root - 网站的目录

**配置补充**

```
server {
    listen 80;
    server_name localhost test.com;
    index index.html
    root /var/www/test;

    # 日志格式设定
    log_format access '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';

    #定义本虚拟主机的访问日志
    access_log  /usr/local/nginx/logs/host.access.log  main;
    access_log  /usr/local/nginx/logs/host.access.404.log  log404;

    # 错误显示页面
    error_page 500 502 503 504 /50x.html
}
```

日志记录对于统计排错来说非常有利的 , 这里记录一组日志相关的配置和常用模式 :

* log\_format - 设置日志格式 .

  * 语法 : log\_format name string
    * name是可以自定义的名字
    * string是拼接的格式
  * 日志格式一般在http层定义 , 可以多定义几套 , 在不同地方调用 , 手机不同内容的日志 . 

变量注释

```
# $remote_add拿到的IP地址是反向代理服务器的iP地址 , $http_x_forwarded_for记录原有客户端的IP地址和原来客户端的请求的服务器地址
$remote_addr , $http_x_forwarded_for - 记录客户端IP地址
$remote_user - 记录客户端用户名称
$request - 记录请求的URL和HTTP协议
$status - 记录请求状态
$body_bytes_sent - 发送给客户端的字节数 , 不包括响应头的大小 ; 该变量与Apache模块mod_log_config里的“%B”参数兼容
$bytes_sent - 发送给客户端的总字节数
$connection - 连接的序列号
$connection_requests - 当前通过一个连接获得的请求数量
$msec - 日志写入时间 . 单位为秒 , 精度是毫秒
$pipe - 如果请求是通过HTTP流水线(pipelined)发送 , pipe值为“p” , 否则为“.”
$http_referer - 记录从哪个页面链接访问过来的
$http_user_agent - 记录客户端浏览器相关信息
$request_length - 请求的长度(包括请求行 , 请求头和请求正文)
$request_time - 请求处理时间 , 单位为秒 , 精度毫秒 ; 从读入客户端的第一个字节开始 , 直到把最后一个字符发送给客户端后进行日志写入为止 .
$time_iso8601 - ISO8601标准格式下的本地时间
$time_local - 通用日志格式下的本地时间
$sent_http_content_range - 发送给客户端的响应头拥有“sent_http_”前缀
$upstream_status - upstream状态
$upstream_addr - 后台upstream的地址 , 即真正提供服务的主机地址
$upstream_response_time - 请求过程中 , upstream响应时间
$ssl_protocol - SSL协议版本
$ssl_cipher - 交换数据中的算法
```

下面是一些例子 , 用来解决不同的问题

```
# 通用日志记录
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"'
                '"$upstream_addr" "$upstream_status" "$upstream_response_time" "$request_time"';

log_format down '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $bytes_sent '
                '"$http_referer" "$http_user_agent" '
                '"$http_range" "$sent_http_content_range"';

log_format access '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" $http_x_forwarded_for';

access_log /usr/local/nginx/logs/main.log main;
access_log /usr/local/nginx/logs/access.log access;
```



