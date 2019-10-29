# Nginx正向代理

```
user www;
worker_processes 1;
error_log /var/log/nginx/error.log debug;

events {
    use epoll;
    worker_connections 1024;
}

http {
    resolver 8.8.8.8;
    server {
        listen 8088;
        location / {
            proxy_pass http://$http_host$request_uri;
        }
    }
}
```

nginx实现代理上网 , 有三个关键点必须注意 , 其余的配置跟普通的nginx一样 :

1. 增加dns解析resolver
2. 增加无server\_name名的server
3. proxy\_pass指令



