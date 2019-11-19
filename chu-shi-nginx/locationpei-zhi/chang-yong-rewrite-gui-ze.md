# 常用Rewrite规则

**规则片段**

```
# 多目录转成参数
# abc.domian.com/sort/2 => abc.domian.com/index.php?act=sort&name=abc&id=2
if ($host ~* (.*)\.domain\.com) {
    set $sub_name $1;   
    rewrite ^/sort\/(\d+)\/?$ /index.php?act=sort&cid=$sub_name&id=$1 last;
}

# 目录对换
# /123456/xxxx -> /xxxx?id=123456
rewrite ^/(\d+)/(.+)/ /$2?id=$1 last;

# 例如下面设定nginx在用户使用ie的使用重定向到/nginx-ie目录下
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /nginx-ie/$1 break;
}

# 目录自动加“/”
if (-d $request_filename){
    rewrite ^/(.*)([^/])$ http://$host/$1$2/ permanent;
}

# 将多级目录下的文件转成一个文件,增强seo效果
# /job-123-456-789.html 指向/job/123/456/789.html
rewrite ^/job-([0-9]+)-([0-9]+)-([0-9]+)\.html$ /job/$1/$2/jobshow_$3.html last;

# 将根目录下某个文件夹指向2级目录
# 如/shanghaijob/ 指向 /area/shanghai/
# 如果你将last改成permanent，那么浏览器地址栏显是 /location/shanghai/
rewrite ^/([0-9a-z]+)job/(.*)$ /area/$1/$2 last;
# 上面例子有个问题是访问/shanghai 时将不会匹配
rewrite ^/([0-9a-z]+)job$ /area/$1/ last;
rewrite ^/([0-9a-z]+)job/(.*)$ /area/$1/$2 last;
# 这样/shanghai 也可以访问了，但页面中的相对链接无法使用，
# 如./list_1.html真实地址是/area /shanghia/list_1.html会变成/list_1.html,导至无法访问
# 那加上自动跳转也是不行咯
# (-d $request_filename)它有个条件是必需为真实目录,所以没有效果
if (-d $request_filename){
    rewrite ^/(.*)([^/])$ http://$host/$1$2/ permanent;
}
# 知道原因后就好办了,手动跳转吧
rewrite ^/([0-9a-z]+)job$ /$1job/ permanent;
rewrite ^/([0-9a-z]+)job/(.*)$ /area/$1/$2 last;
# 文件和目录不存在的时候重定向：
if (!-e $request_filename) {
    proxy_pass http://127.0.0.1;
}
```

**IP限制与域名跳转**

```
# 只充许固定ip访问网站,并加上密码
# 这里的加上密码还可以给nginx_status加上密码访问
# htpasswd文件的内容可以用apache提供的htpasswd工具来产生
root  /opt/htdocs/www;
allow   208.97.167.194;
allow   222.33.1.2;
allow   231.152.49.4;
deny    all;
auth_basic "C1G_ADMIN";
auth_basic_user_file htpasswd;

# 域名跳转
server
{
    listen       80;
    server_name  jump.test.com;
    index index.html index.htm index.php;
    root  /opt/htdocs/www;
    rewrite ^/ http://www.test.com/;
    access_log  off;
}

# 多域名转向,比如例子中有com和net两个域名,配置让net域名转到com上
# 301重定向
server_name  www.test.com www.test.net;
index index.html index.htm index.php;
root  /opt/www/htdocs;
if ($host ~ "test\.net") {
    rewrite ^(.*) http://www.test.com$1 permanent;
}

# 三级域名跳转
if ($http_host ~* "^(.*)\.i\.test\.com$") {
    rewrite ^(.*) http://top.test.com$1;
    break;
}

# 域名镜向
server
{
    listen       80;
    server_name  mirror.test.com;
    index index.html index.htm index.php;
    root  /opt/htdocs/www;
    rewrite ^/(.*) http://www.test.com/$1 last;
    access_log  off;
}
```

**框架网站使用**

```
# thinkphp
include /usr/local/nginx/conf/rewrite/thinkphp.conf;
location / {
  if (!-e $request_filename) {
    rewrite ^(.*)$ /index.php?s=$1 last;
    break;
  }
}

location ~ \.php {
  #fastcgi_pass remote_php_ip:9000;
  fastcgi_pass unix:/dev/shm/php-cgi.sock;
  fastcgi_index index.php;
  include fastcgi_params;
  set $real_script_name $fastcgi_script_name;
  if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
    set $real_script_name $1;
    #set $path_info $2;
  }
  fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
  fastcgi_param SCRIPT_NAME $real_script_name;
  #fastcgi_param PATH_INFO $path_info;
}

# laravel
include /usr/local/nginx/conf/rewrite/laravel.conf;
location / {
  try_files $uri $uri/ /index.php?$query_string;
}

# wordpress
include /usr/local/nginx/conf/rewrite/wordpress.conf;
location / {
  try_files $uri $uri/ /index.php?$args;
}
rewrite /wp-admin$ $scheme://$host$uri/ permanent;

# Discuz
include /usr/local/nginx/conf/rewrite/discuz.conf;
rewrite ^([^\.]*)/topic-(.+)\.html$ $1/portal.php?mod=topic&topic=$2 last;
rewrite ^([^\.]*)/article-([0-9]+)-([0-9]+)\.html$ $1/portal.php?mod=view&aid=$2&page=$3 last;
rewrite ^([^\.]*)/forum-(\w+)-([0-9]+)\.html$ $1/forum.php?mod=forumdisplay&fid=$2&page=$3 last;
rewrite ^([^\.]*)/thread-([0-9]+)-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=viewthread&tid=$2&extra=page%3D$4&page=$3 last;
rewrite ^([^\.]*)/group-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=group&fid=$2&page=$3 last;
rewrite ^([^\.]*)/space-(username|uid)-(.+)\.html$ $1/home.php?mod=space&$2=$3 last;
rewrite ^([^\.]*)/blog-([0-9]+)-([0-9]+)\.html$ $1/home.php?mod=space&uid=$2&do=blog&id=$3 last;
rewrite ^([^\.]*)/(fid|tid)-([0-9]+)\.html$ $1/index.php?action=$2&value=$3 last;
rewrite ^([^\.]*)/([a-z]+[a-z0-9_]*)-([a-z0-9_\-]+)\.html$ $1/plugin.php?id=$2:$3 last;
#if (!-e $request_filename) {
#  return 404;
#}

# Typecho
include /usr/local/nginx/conf/rewrite/typecho.conf;
if (!-e $request_filename) {
  rewrite ^(.*)$ /index.php$1 last;
}

# Ecshop
include /usr/local/nginx/conf/rewrite/ecshop.conf;
if (!-e $request_filename) {
  rewrite "^/index\.html" /index.php last;
  rewrite "^/category$" /index.php last;
  rewrite "^/feed-c([0-9]+)\.xml$" /feed.php?cat=$1 last;
  rewrite "^/feed-b([0-9]+)\.xml$" /feed.php?brand=$1 last;
  rewrite "^/feed\.xml$" /feed.php last;
  rewrite "^/category-([0-9]+)-b([0-9]+)-min([0-9]+)-max([0-9]+)-attr([^-]*)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$" /category.php?id=$1&brand=$2&price_min=$3&price_max=$4&filter_attr=$5&page=$6&sort=$7&order=$8 last;
  rewrite "^/category-([0-9]+)-b([0-9]+)-min([0-9]+)-max([0-9]+)-attr([^-]*)(.*)\.html$" /category.php?id=$1&brand=$2&price_min=$3&price_max=$4&filter_attr=$5 last;
  rewrite "^/category-([0-9]+)-b([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$" /category.php?id=$1&brand=$2&page=$3&sort=$4&order=$5 last;
  rewrite "^/category-([0-9]+)-b([0-9]+)-([0-9]+)(.*)\.html$" /category.php?id=$1&brand=$2&page=$3 last;
  rewrite "^/category-([0-9]+)-b([0-9]+)(.*)\.html$" /category.php?id=$1&brand=$2 last;
  rewrite "^/category-([0-9]+)(.*)\.html$" /category.php?id=$1 last;
  rewrite "^/goods-([0-9]+)(.*)\.html" /goods.php?id=$1 last;
  rewrite "^/article_cat-([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$" /article_cat.php?id=$1&page=$2&sort=$3&order=$4 last;
  rewrite "^/article_cat-([0-9]+)-([0-9]+)(.*)\.html$" /article_cat.php?id=$1&page=$2 last;
  rewrite "^/article_cat-([0-9]+)(.*)\.html$" /article_cat.php?id=$1 last;
  rewrite "^/article-([0-9]+)(.*)\.html$" /article.php?id=$1 last;
  rewrite "^/brand-([0-9]+)-c([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)\.html" /brand.php?id=$1&cat=$2&page=$3&sort=$4&order=$5 last;
  rewrite "^/brand-([0-9]+)-c([0-9]+)-([0-9]+)(.*)\.html" /brand.php?id=$1&cat=$2&page=$3 last;
  rewrite "^/brand-([0-9]+)-c([0-9]+)(.*)\.html" /brand.php?id=$1&cat=$2 last;
  rewrite "^/brand-([0-9]+)(.*)\.html" /brand.php?id=$1 last;
  rewrite "^/tag-(.*)\.html" /search.php?keywords=$1 last;
  rewrite "^/snatch-([0-9]+)\.html$" /snatch.php?id=$1 last;
  rewrite "^/group_buy-([0-9]+)\.html$" /group_buy.php?act=view&id=$1 last;
  rewrite "^/auction-([0-9]+)\.html$" /auction.php?act=view&id=$1 last;
  rewrite "^/exchange-id([0-9]+)(.*)\.html$" /exchange.php?id=$1&act=view last;
  rewrite "^/exchange-([0-9]+)-min([0-9]+)-max([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$" /exchange.php?cat_id=$1&integral_min=$2&integral_max=$3&page=$4&sort=$5&order=$6 last;
  rewrite "^/exchange-([0-9]+)-([0-9]+)-(.+)-([a-zA-Z]+)(.*)\.html$" /exchange.php?cat_id=$1&page=$2&sort=$3&order=$4 last;
  rewrite "^/exchange-([0-9]+)-([0-9]+)(.*)\.html$" /exchange.php?cat_id=$1&page=$2 last;
  rewrite "^/exchange-([0-9]+)(.*)\.html$" /exchange.php?cat_id=$1 last;
}
```



