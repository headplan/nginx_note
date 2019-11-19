# 配置用例

**nginx.conf全局配置文件**

```
user www www; # 工作进程运行的用户及用户组
worker_processes auto; # 开启的工作进程数,建议设置为等于CPU总核心数
# [ debug | info | notice | warn | error | crit ]
error_log logs/error.log crit; # 全局错误日志输出位置以及级别
pid logs/nginx.pid; # 存储Nginx进程id的文件路径
worker_rlimit_nofile 51200; # 一个进程打开文件数限制,但受Linux的此参数限制

# 工作模式及连接数上线
events {
  use epoll; # 工作模式,如果是MacOS使用kqueue模式
  worker_connections 51200; # 每个进程最大连接数(进程就是worker_processes)
  multi_accept on; 设置Nginx是否允许,在已经得到一个新连接的通知时,接收尽可能更多的连接
}

# http配置
http {
  # 添加apk和ipa后缀的文件类型
  # application/vnd.android.package-archive apk;
  # application/iphone pxl ipa;
  include mime.types; # 文件类型
  default_type application/octet-stream; # 默认二进制流
  # charset utf-8; # 默认编码
  # autoindex on; # 开启目录列表访问,合适下载服务器,默认关闭

  # 服务器客户大小以及缓冲器大小等配置
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 1024m;
  client_body_buffer_size 10m;

  # 高效传输文件模式
  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 60;
  server_tokens off;

  # fastcgi模块优化配置
  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  fastcgi_intercept_errors on;
  # 开启fastcgi缓存
  fastcgi_cache TEST; 
  fastcgi_cache_valid 200 302 1h; 
  fastcgi_cache_valid 301 1d; 
  fastcgi_cache_valid any 1m;

  # gzip模块配置
  gzip on;
  gzip_buffers 16 8k;
  gzip_comp_level 6;
  gzip_http_version 1.1;
  gzip_min_length 256;
  gzip_proxied any;
  gzip_vary on;
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";

  # 静态文件缓存配置
  open_file_cache max=1000 inactive=20s;
  open_file_cache_valid 30s;
  open_file_cache_min_uses 2;
  open_file_cache_errors on;

  # 开启限制IP连接数的时候需要使用
  # limit_zone crawler $binary_remote_addr 10m; # 一般用在限制单线下载以及限速下载时使用

  # 日志记录
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

  # 负载均衡配置(参考相关笔记)

  # ===== Defaule Server =====
  server {
    listen 80;
    server_name localhost test.com;
    access_log /usr/local/nginx/logs/access.log combined;
    index index.html index.htm index.php;
    root /var/www/test;
    # if ($host != test.com) {  return 301 $scheme://test.com$request_uri;  }
    include /usr/local/nginx/conf/rewrite/other.conf;
    # error_page 404 /404.html;
    # error_page 502 /502.html;

    # 反向代理配置(参考相关笔记) 

    location /nginx_status {
      stub_status on;
      access_log off;
      allow 127.0.0.1;
      deny all;
      # auth_basic "NginxStatus";
      # auth_basic_user_file htpasswd;
    }

    location ~ .*\.(wma|wmv|asf|mp3|mmf|zip|rar|jpg|gif|png|swf|flv|mp4)$ {
      valid_referers none blocked *.test.com www.test.com;
      if ($invalid_referer) {
        rewrite ^/ http://test.test.com/test.gif;
        return 412;
      }
    }

    location ~ [^/]\.php(/|$) {
      #fastcgi_pass remote_php_ip:9000;
      fastcgi_pass unix:/dev/shm/php-cgi.sock;
      fastcgi_index index.php;
      include fastcgi.conf;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico)$ {
      expires 30d;
      access_log off;
    }

    location ~ .*\.(js|css)?$ {
      expires 7d;
      access_log off;
    }

    location ~ /\.ht {
      deny all;
    }

  }

  # ===== vhost =====
  include vhost/*.conf;
}
```



