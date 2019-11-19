# 常用location配置

**PHP常用配置**

```
# 防盗链配置
location ~ .*\.(wma|wmv|asf|mp3|mmf|zip|rar|jpg|gif|png|swf|flv|mp4)$ {
    valid_referers none blocked *.test.com www.test.com;
    if ($invalid_referer) {
            rewrite ^/ /403.html;
            return 403;
    }
}
# 转发动态请求到后端应用服务器
# 这里直接走内存里的unix:/dev/shm/php-cgi.sock;当然还要修改php-fpm.conf的配置.
# 引入了fastcgi.conf配置,所以这里就不用添加下面这个配置了
# fastcgi_param SCRIPT_FILENAME $fastcgi_script_name
location ~ [^/]\.php(/|$) {
    #fastcgi_pass remote_php_ip:9000;
    fastcgi_pass unix:/dev/shm/php-cgi.sock;
    fastcgi_index index.php;
    include fastcgi.conf;
}
# 处理静态文件请求
# 关闭了日志记录
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico)$ {
    expires 30d;
    access_log off;
}
# 处理静态文件请求
# 关闭了日志记录
location ~ .*\.(js|css)?$ {
    expires 7d;
    access_log off;
}
# 禁止htaccess
location ~ /\.ht {
    deny all;
}
```

**开启性能统计**

```
# 编译源码时带上参数"--with-http_stub_status_module",就安装了Nginx的统计模块
# 配置,其中allow只允许本机访问,deny禁用所有,可以注释打开
# 还可以配置加密访问
# htpasswd文件的内容可以用apache提供的htpasswd工具来产生
location /nginx_status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    deny all;
    # auth_basic "NginxStatus";
    # auth_basic_user_file htpasswd;
}
# 访问显示内容
Active connections: 1 
server accepts handled requests
 598 598 578 
Reading: 0 Writing: 1 Waiting: 0 

Active connections - 当前nginx正在处理的活动链接数
server accepts handled requests - 共处理了598次连接,598次握手,578次请求
Reading - 读取到客户端的header信息数
writing - 返回给客户端的header信息数
waiting - 开启keep-alive的情况下,这个值等于Active-(Reading+Writing),是已经处理完成,等待下次请求指令的驻留连接.
当快速请求完毕时,Waiting数比较多是正常的;Reading+Writing比较多,则说明后台并发访问量较大,正在处理过程中.
```

**禁止配置**

```
# 禁止htaccess
location ~/\.ht {
    deny all;
}
# 禁止多个目录
location ~ ^/(cron|templates)/ {
    deny all;
    break;
}

# 禁止以/data开头的文件
# 可以禁止/data/下多级目录下.log.txt等请求
location ~ ^/data {
    deny all;
}

# 禁止单个目录
# 不能禁止.log.txt能请求
location /searchword/cron/ {
    deny all;
}

# 禁止单个文件
location ~ /data/sql/data.sql {
    deny all;
}
```

**过期时间与防盗链**

```
# 给favicon.ico和robots.txt设置过期时间;
# 这里为favicon.ico为99天,robots.txt为7天;
# 并不记录404错误日志.
location ~(favicon.ico) {
    log_not_found off;
    expires 99d;
    break;
}

location ~(robots.txt) {
    log_not_found off;
    expires 7d;
    break;
}

# 设定某个文件的过期时间;
# 这里为600秒,并不记录访问日志
location ^~ /html/scripts/loadhead_1.js {
    access_log   off;
    root /opt/lampp/htdocs/web;
    expires 600;
    break;
}

# 文件反盗链并设置过期时间
# 这里的return 412为自定义的http状态码,默认为403,方便找出正确的盗链的请求
# "rewrite ^/ http://test.test.com/test.gif;" 显示一张防盗链图片
# "access_log off;" 不记录访问日志,减轻压力
# "expires 3d;" 所有文件3天的浏览器缓存
location ~* ^.+\.(jpg|jpeg|gif|png|swf|rar|zip|css|js)$ {
    valid_referers none blocked *.test.com *.test.net localhost 192.169.1.11;
    if ($invalid_referer) {
        rewrite ^/ http://test.test.com/test.gif
        return 412;
        break;
    }
    access_log off;
    root /opt/www/htdocs/web;
    expires 3d;
    break;
}
```



