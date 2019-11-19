# 常用配置

Nginx的配置文件nginx.conf是纯文本文件 , 位于Nginx安装目录的conf目录下 , 整个配置文件是以块的形式组织 , 每个块以{}来表示 . 采用嵌套的方式 , 一个大块中可以包括小块 . 最大的块是main块 , main块里包含event块和http块 , http块包含了upstream块和server块 , server块包含了多个location块 .

* main - Nginx的全局属性配置
* event - Nginx的工作模式及连接数上线
* http - http服务器相关属性的配置
* upstream - 负载均衡属性的配置
* server - 虚拟主机的配置
* location - location的配置

![](/assets/peizhitu.png)

Nginx的全局配置

```
user www www;
worker_processes 4;
error_log /home/wwwlogs/nginx_error.log crit;
pid /usr/local/nginx/logs/nginx.pid;
worker_rlimit_nofile 52000;
```

* user - 指定了Nginx工作进程运行的用户及用户组 , 默认是nobody , 这里配置的是用户www和用户组www
* worker\_processes - 指定Nginx开启的工作进程数 . 每个进程大约占用10-12MB的内存 . 如果是多核的CPU , 这里应设置和CPU核数一样的进程数 . 
* error\_log - 全局错误日志的位置与日志输出的级别 . 日志的输出级别可选择 . 其中debug级别输出的日志最详细 . 
  * debug
  * info
  * notice
  * warn
  * error
  * crit
* pid - 存储Nginx进程id的文件路径
* `worker_rlimit_nofile`- 指定了一个Nginx进程最多可以打开的文件描述符 . 这里的配置受限于Linux中的那个配置 , 前面我们提到过 , 就是Too many open files那个错误 .  

#### event配置

```
events
{
    use epoll;
    work_connections 51200;
    multi_accept on;
}
```

* use - 指定Nginx的工作模式 , 前面已经提过 , epoll是首选 , FreeBSD下kqueue是首选
  * select
  * poll
  * kqueue
  * epoll
  * rtsig
  * /dev/poll
* worker\__connections - 定义每个worker pricess的最大连接数 , 默认1024 . 这里的配置也受限于Linux中最多可以打开的文件描述符数限制 . 所以Nginx可以处理的最大连接数为max_\_clients = worker\_processes × worker\_connections
* multi\_accept - 设置Nginx是否允许,在已经得到一个新连接的通知时,接收尽可能更多的连接 . 

#### http配置

```
http
{
    include mime.types;
    default_type Application/octet-stream;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 50m;
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 60;
    tcp_nodelay on;
    server_tokens off;

    # autoindex on;

    # fastgci模块优化

    # gzip模块配置

    include vhost/*.conf;
}
```

* include - 包含其他的配置文件 , 一般是mime.types一个文件类型的列表 , 和nginx.conf配置文件同级目录下
* default\_type - 默认类型application/octet-stream;二进制流格式 . 意思是当文件类型未定义时 , 使用二进制流的格式.
* server\_names\_hash\_bucket\_size - 服务器名字的hash表大小 . 一般设置一个很长的域名之后会提示修改这个配置大小的错误提示
* > 保存服务器名字的hash表是由指令server\_names\_hash\_max\_size 和server\_names\_hash\_bucket\_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.
* client\_header\_buffer\_size - 客户端请求头buffersize的大小
* large\_client\_header\_buffer - 客户端请求中较大的消息头的缓存的数量和大小 , 这里第一个参数是数量 , 第二个参数是大小
* client\_max\_body\_size - 客户端请求中http body的大小 , 一般可以理解为请求的文件大小 . 
* client\_body\_buffer\_size - 配置上一个配置的缓冲器大小 , 可以理解为上面的配置的分页大小
* sendfile - 是否启动高效传输文件的模式 . 
  * sendfile可以让Nginx在传输文件时直接在磁盘和tcp Socket之间传输数据 . 如果这个参数不开启 , 会先在用户空间申请一个buffer缓冲器 , 用read函数把数据从磁盘读到cache , 再从cache读取到用户空间的buffer , 再用write函数把数据从用户空间的buffer写入到内核的buffer , 最后到TCP Socket . 开始这个参数就可以让数据不用经过用户buffer了 . 
    ![](/assets/jinchengkongjian.png)
* tcp\_nopush - 仅在sendfile开启时有效 , 主要防止网络堵塞 . 告诉Nginx在一个数据包里发送所有头文件 , 而不一个接一个的发送 . 
* tcp\_nodelay - 仅在sendfile开启时有效 . 告诉nginx不要缓存数据 , 而是一段一段的发送 , 当需要及时发送数据时 . 
* keepalive\_timeout - 设置客户端保持活动连接的时间 . 超时服务器会关闭连接 . 
* server\_tokens - 建议off , 这里是关闭返回Nginx的版本信息 . 
* autoindex  - 开启目录列表访问 , 合适下载服务器 , 默认关闭 . 

> **fastcgi模块参考下一篇文章**

**补充说明内容**

> **这些内容是前面提到过的配置的解释说明帮助理解**

```
#指定进程可以打开的最大描述符：数目
#工作模式与连接数上限
#这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
#现在在linux 2.6内核下开启文件打开数为65535，worker_rlimit_nofile就相应应该填写65535。
#这是因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。

worker_rlimit_nofile 65535;

#单个进程最大连接数（最大连接数=连接数*进程数）
#根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为。

worker_connections 65535;

===================================

#客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。
#分页大小可以用命令getconf PAGESIZE 取得。
#[root@web001 ~]# getconf PAGESIZE
#4096
#但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。

client_header_buffer_size 4k;

#服务器名字的hash表大小
#保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.

server_names_hash_bucket_size 128;

#客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。

client_header_buffer_size 32k;

#客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。

large_client_header_buffers 4 64k;

#设定通过nginx上传文件的大小

client_max_body_size 8m;

#开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
#sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。

sendfile on;
```

**gzip模块配置**

Nginx的httpgzip模块 , 支持在线实时压缩输出数据流 . 编译Nginx时需要带上参数

`--with-http_gzip_static_module`才能使用这个模块 .

```
gzip on; # 开启gzip压缩输出
gzip_min_length 1k; # 最小压缩文件大小
gzip_buffers 4 16k; # 压缩缓冲区
gzip_http_version 1.0; # 支持的HTTP协议版本(默认1.1,前端如果是squid2.5请使用1.0)
gzip_comp_level 2; # 压缩等级
# 压缩类型,默认就已经包含textml,所以下面就不用再写了,写上去也不会有问题,但是会有一个warn
gzip_types text/plain application/x-javascript text/css application/xml;
gzip_vary on; # 是否让前端缓存缓存服务器缓存压缩后的GZIP文件
```

这里挑了几个解释一下 :

* gzip\_min\_length - 设置只有当页面的大小大于这个值时 , 才启用gzip压缩 . 页面大小值通过读取http头"Content-Length"来获取 . 建议是1KB , 文件太小 , 压缩有有可能会更大 .
* gzip\_buffers - gzip的缓冲区的数量和大小 . 默认是申请和"Content-Length"中一样大小的缓冲区 .  
* gzip\_comp\_level - 压缩等级 . 取值1-9 . 1是压缩比最低 , 但最快 . 9是压缩比最高, 最慢 , 而且特别消耗CPU资源 . 

补充说明

> **gzip\_proxied**
>
> Nginx做为反向代理的时候启用 , 参数 :
>
> param:off\|expired\|no-cache\|no-sotre\|private\|no\_last\_modified\|no\_etag\|auth\|any\]
>
> **expample:gzip\_proxied no-cache;**
>
> * off – 关闭所有的代理结果数据压缩
> * expired – 启用压缩，如果header中包含”Expires”头信息
> * no-cache – 启用压缩，如果header中包含”Cache-Control:no-cache”头信息
> * no-store – 启用压缩，如果header中包含”Cache-Control:no-store”头信息
> * private – 启用压缩，如果header中包含”Cache-Control:private”头信息
> * no\_last\_modified – 启用压缩，如果header中包含”Last\_Modified”头信息
> * no\_etag – 启用压缩，如果header中包含“ETag”头信息
> * auth – 启用压缩，如果header中包含“Authorization”头信息
> * any – 无条件压缩所有结果数据
>
> **gzip\_disable "MSIE \[1-6\].\(?!.\*SV1\)";**  
> 由于IE6及以下版本对Gzip的支持不够完善 , 且可能造成浏览器的假死 , 因而通常禁用IE6以下的Gzip压缩功能 .

**静态文件缓存配置**

```
open_file_cache max=100000 inactive=20s;
open_file_cache_valid 30s;
open_file_cache_min_uses 2; 
open_file_cache_errors on;
```

* open\_file\_cache - 打开缓存的同时也指定了缓存最大数目 , 以及缓存的时间 . 我们可以设置一个相对高的最大时间 , 这样我们可以在它们不活动超过20秒后清除掉
* open\_file\_cache\_valid - 这个是指多长时间检查一次缓存的有效信息 . 
* open\_file\_cache\_min\_uses - open\_file\_cache指令中的inactive参数时间内文件的最少使用次数 , 如果超过这个数字 , 文件描述符一直是在缓存中打开的 , 如上例 , 如果有一个文件在inactive时间内一次没被使用 , 它将被移除 . 
* open\_file\_cache\_errors - 指定是否在搜索一个文件时记录cache错误 . 



