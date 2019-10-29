# try\_files

Nginx的配置语法灵活 , 可控制度非常高 . 在0.7以后的版本中加入了一个try\_files指令 , 配合命名location , 可以部分替代原本常用的rewrite配置方式 , 提高解析效率 .

### try\_files指令

语法：try\_files file ... uri 或 try\_files file ... = code  
默认值：无  
作用域：server location

其作用是按顺序检查文件是否存在，返回第一个找到的文件或文件夹（结尾加斜线表示为文件夹），如果所有的文件或文件夹都找不到，会进行一个内部重定向到最后一个参数。

需要注意的是，只有最后一个参数可以引起一个内部重定向，之前的参数只设置内部URI的指向。最后一个参数是回退URI且必须存在，否则会出现内部500错误。命名的location也可以使用在最后一个参数中。与rewrite指令不同，如果回退URI不是命名的location那么$args不会自动保留，如果你想保留$args，则必须明确声明。

**常见错误**

try\_files 按顺序检查文件是否存在，返回第一个找到的文件，至少需要两个参数，但最后一个是内部重定向也就是说和rewrite效果一致，前面的值是相对$document\_root的文件路径。也就是说参数的意义不同，甚至可以用一个状态码 \(404\)作为最后一个参数。如果不注意会有死循环造成500错误。

示例1：

```
location ~.*\.(gif|jpg|jpeg|png)$ {
        root /web/wwwroot;
        try_files /static/$uri $uri;
}
```

原意图是访问[http://example.com/test.jpg](http://example.com/test.jpg) 时先去检查/web/wwwroot/static/test.jpg是否存在，不存在就取/web/wwwroot/test.jpg

但由于最后一个参数是一个内部重定向，所以并不会检查/web/wwwroot/test.jpg是否存在，只要第一个路径不存在就会重新向然后再进入这个location造成死循环。结果出现500 Internal Server Error

实例2:

```
    location ~.*\.(gif|jpg|jpeg|png)$ {
        root /web/wwwroot;
        try_files /static/$uri $uri 404;
    }
```

这样才会先检查/web/wwwroot/static/test.jpg是否存在，不存在就取/web/wwwroot/test.jpg 再不存在则返回404 not found

最后：

可能你会觉得500和404好像都是错误码，差别不大，其实500不仅多次重定向浪费性能，还会影响到nginx的upstream造成错误的判断当前机器有故障而将新请求路由到其它机器甚至备用机，404则不会。

