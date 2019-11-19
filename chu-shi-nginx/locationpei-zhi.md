# location配置

location支持正则表达式和条件判断匹配 , 用户可以通过location指令对动态,静态网页进行过滤 .

例如 :

```
location ~ .*\.(git|jpg|jpeg|png)$
{
    expires 30d;
}
```

这段location的配置的含义是 : 经过正则表达式匹配 , 设置文件格式为GIT,JPEG,PNG的文件在HTTP应答中"Expires"和"Cache-Control"的HTTP头 , 以达到在浏览器中缓存图片的作用 , 大概意思就是把图片在浏览器中缓存30天 .

语法规则： location \[=\|~\|~\*\|^~\] /uri/ { … }

* = 开头表示精确匹配
* ^~ 开头表示uri以某个常规字符串开头 , 理解为匹配url路径即可 . nginx不对url做编码 , 因此请求为/static/20%/aa , 可以被规则^~ /static/ /aa匹配到\(注意是空格\)
* ~ 开头表示区分大小写的正则匹配
* ~\*  开头表示不区分大小写的正则匹配
* !~和!~\*分别为区分大小写不匹配及不区分大小写不匹配的正则
* / 通用匹配 , 任何请求都会匹配到

多个location配置的情况下匹配顺序为 :

首先匹配 = , 其次匹配^~ , 其次是按文件中顺序的正则匹配 , 最后是交给 / 通用匹配 . 当有匹配成功时候 , 停止匹配 , 按当前匹配规则处理请求 .

```
# 例子
location = / {  
   #规则A  
}  
location = /login {  
   #规则B  
}  
location ^~ /static/ {  
   #规则C  
}  
location ~ \.(gif|jpg|png|js|css)$ {  
   #规则D  
}  
location ~* \.png$ {  
   #规则E  
}  
location !~ \.xhtml$ {  
   #规则F  
}  
location !~* \.xhtml$ {  
   #规则G  
}  
location / {  
   #规则H  
}
```

上文例子效果描述 :

* 访问根目录/ , 比如[http://localhost/将匹配规则A](http://localhost/将匹配规则A)
* 访问 [http://localhost/login将匹配规则B](http://localhost/login将匹配规则B) , [http://localhost/register则匹配规则H](http://localhost/register则匹配规则H)
* 访问 [http://localhost/static/a.html](http://localhost/static/a.html) 将匹配规则C
* 访问 [http://localhost/a.gif](http://localhost/a.gif) , [http://localhost/b.jpg](http://localhost/b.jpg) 将匹配规则D和规则E , 但是规则D顺序优先 , 规则E不起作用 , 而 [http://localhost/static/c.png](http://localhost/static/c.png) 则优先匹配到规则C
* 访问 [http://localhost/a.PNG](http://localhost/a.PNG) 则匹配规则E , 而不会匹配规则D , 因为规则E不区分大小写
* 访问 [http://localhost/a.xhtml](http://localhost/a.xhtml) 不会匹配规则F和规则G , [http://localhost/a.XHTML不会匹配规则G](http://localhost/a.XHTML不会匹配规则G) , 因为不区分大小写 . 规则F , 规则G属于排除法 , 符合匹配规则但是不会匹配到 , 所以想想看实际应用中哪里会用到
* 访问 [http://localhost/category/id/1111](http://localhost/category/id/1111) 则最终匹配到规则H , 因为以上规则都不匹配 , 这个时候应该是nginx转发请求给后端应用服务器 , 比如FastCGI\(php\) , tomcat\(jsp\) , nginx作为方向代理服务器存在



