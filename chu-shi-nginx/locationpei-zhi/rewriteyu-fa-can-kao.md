# Rewrite语法参考

```
last – 基本上都用这个Flag.停止当前这个请求,并根据rewrite匹配的规则重新发起一个请求.新请求又从第一阶段开始执行.
break – 中止Rewirte,不在继续匹配.相对last,break并不会重新发起一个请求,只是跳过当前的rewrite阶段,并执行本请求后续的执行阶段.
redirect – 返回临时重定向的HTTP状态302
permanent – 返回永久重定向的HTTP状态301

注：last和break最大的不同在于
- break是终止当前location的rewrite检测,而且不再进行location匹配 - last是终止当前location的rewrite检测,但会继续重试location匹配并处理区块中的rewrite规则

1、下面是可以用来判断的表达式：
-f和!-f用来判断是否存在文件
-d和!-d用来判断是否存在目录
-e和!-e用来判断是否存在文件或目录
-x和!-x用来判断文件是否可执行

2、下面是可以用作判断的全局变量
$args #这个变量等于请求行中的参数。
$content_length #请求头中的Content-length字段。
$content_type #请求头中的Content-Type字段。
$document_root #当前请求在root指令中指定的值。
$host #请求主机头字段，否则为服务器名称。
$http_user_agent #客户端agent信息
$http_cookie #客户端cookie信息
$limit_rate #这个变量可以限制连接速率。
$request_body_file #客户端请求主体信息的临时文件名。
$request_method #客户端请求的动作，通常为GET或POST。
$remote_addr #客户端的IP地址。
$remote_port #客户端的端口。
$remote_user #已经经过Auth Basic Module验证的用户名。
$request_filename #当前请求的文件路径，由root或alias指令与URI请求生成。
$query_string #与$args相同。
$scheme #HTTP方法（如http，https）。
$server_protocol #请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
$server_addr #服务器地址，在完成一次系统调用后可以确定这个值。
$server_name #服务器名称。
$server_port #请求到达服务器的端口号。
$request_uri #包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
$uri #不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
$document_uri #与$uri相同。
例：http://localhost:88/test1/test2/test.php
$host：localhost
$server_port：88
$request_uri：http://localhost:88/test1/test2/test.php
$document_uri：/test1/test2/test.php
$document_root：D:\nginx/html
$request_filename：D:\nginx/html/test1/test2/test.php
```



