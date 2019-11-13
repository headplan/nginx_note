# 编译Nginx

直接安装Nginx的二进制文件 , 会把模块直接编译进来 , 而官方模块并不是默认都开启的 , 如果想添加第三方模块 , 必须通过编译Nginx , 才能将第三方生态圈的功能添加到Nginx中 .

#### 下载Nginx

[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

选择Stable version稳定版本 :

[http://nginx.org/download/nginx-1.16.1.tar.gz](http://nginx.org/download/nginx-1.16.1.tar.gz)

**解压压缩包**

```
tar -xzvf ./nginx-1.16.1.tar.gz
```

**目录结构**

```
./nginx-1.16.1
├── CHANGES - 每个Nginx版本提供的特性feature,change和bugfix
├── CHANGES.ru - 俄语版
├── LICENSE - 软件许可证
├── README - Readme文件
├── auto/
├── conf/
├── configure*
├── contrib/
├── html/
├── man/
└── src/
```

**auto目录**

除了标注的目录 , 其他文件及目录都是为了辅助configure脚本执行时 , 去判断nginx支持哪些模块 , 当前的操作系统有什么样的特性可以给Nginx使用 .

```
├── auto/
│   ├── cc/ - 用于编译的
│   ├── define
│   ├── endianness
│   ├── feature
│   ├── have
│   ├── have_headers
│   ├── headers
│   ├── include
│   ├── init
│   ├── install
│   ├── lib/ - lib库
│   ├── make
│   ├── module
│   ├── modules
│   ├── nohave
│   ├── options
│   ├── os/ - 对所有操作系统的判断
│   ├── sources
│   ├── stubs
│   ├── summary
│   ├── threads
│   ├── types/
│   └── unix
```

**conf目录**

示例配置文件的目录

```
./conf
├── fastcgi.conf
├── fastcgi_params
├── koi-utf
├── koi-win
├── mime.types
├── nginx.conf
├── scgi_params
├── uwsgi_params
└── win-utf
```

**configure脚本**

是一个用来生成中间文件 , 执行编译前的必备动作 .

**contrib目录**

提供了两个perl脚本 , 和vim的配置\(可以高亮nginx配置\) .

```
contrib
├── README
├── geo2nginx.pl
├── unicode2nginx
│   ├── koi-utf
│   ├── unicode-to-nginx.pl
│   └── win-utf
└── vim
    ├── ftdetect
    │   └── nginx.vim
    ├── ftplugin
    │   └── nginx.vim
    ├── indent
    │   └── nginx.vim
    └── syntax
        └── nginx.vim
```

**html目录**

```
./html
├── 50x.html - 50x错误页面
└── index.html - 欢迎页面
```

**man目录**

Linux对nginx的帮助文件 .

```
man ./man/nginx.8
```

**src目录**

源代码目录 .

```
./src
├── core/
├── event/
├── http/
├── mail/
├── misc/
├── os/
└── stream/
```

#### 开始编译

**查看编译参数**

```
./configure --help | more
```

**设置文件 , 模块路径**

默认指定--prefix参数即可 .

```
--prefix=PATH                      set installation prefix
--sbin-path=PATH                   set nginx binary pathname
--modules-path=PATH                set modules path
--conf-path=PATH                   set nginx.conf pathname
--error-log-path=PATH              set error log pathname
--pid-path=PATH                    set nginx.pid pathname
--lock-path=PATH                   set nginx.lock pathname
```

**确定使用和不使用哪些模块**

```
--with-select_module               enable select module
--without-select_module            disable select module
--with-poll_module                 enable poll module
--without-poll_module              disable poll module
```

**还有一些特殊的参数**

```
--with-cc=PATH                     set C compiler pathname
--with-cpp=PATH                    set C preprocessor pathname
--with-cc-opt=OPTIONS              set additional C compiler options
--with-ld-opt=OPTIONS              set additional linker options
--with-cpu-opt=CPU                 build for the specified CPU, valid values:
                                   pentium, pentiumpro, pentium3, pentium4,
                                   athlon, opteron, sparc32, sparc64, ppc64
```

**开始编译**

```
./configure --prefix=/Users/mynginx
```

如果没有报错 , 说明Nginx已经编译成功了 . 现在所有nginx的配置 , 以及运行时的目录都会列出来

```
nginx path prefix: "/Users/mynginx"
nginx binary file: "/Users/mynginx/sbin/nginx"
nginx modules path: "/Users/mynginx/modules"
nginx configuration prefix: "/Users/mynginx/conf"
nginx configuration file: "/Users/mynginx/conf/nginx.conf"
nginx pid file: "/Users/mynginx/logs/nginx.pid"
nginx error log file: "/Users/mynginx/logs/error.log"
nginx http access log file: "/Users/mynginx/logs/access.log"
nginx http client request body temporary files: "client_body_temp"
nginx http proxy temporary files: "proxy_temp"
nginx http fastcgi temporary files: "fastcgi_temp"
nginx http uwsgi temporary files: "uwsgi_temp"
nginx http scgi temporary files: "scgi_temp"
```

在./configure完成之后 , 会生成一些中间文件 , 在objs目录下 .

```
./objs
├── Makefile
├── autoconf.err
├── ngx_auto_config.h
├── ngx_auto_headers.h
├── ngx_modules.c
└── src/
```

这里最重要的是ngx\_modules.c文件 , 它决定了接下来执行编译时 , 有哪些模块 , 会被编译进Nginx .

```c
extern ngx_module_t  ngx_core_module;
extern ngx_module_t  ngx_errlog_module;
extern ngx_module_t  ngx_conf_module;
extern ngx_module_t  ngx_regex_module;
extern ngx_module_t  ngx_events_module;
extern ngx_module_t  ngx_event_core_module;
extern ngx_module_t  ngx_kqueue_module;
extern ngx_module_t  ngx_http_module;
extern ngx_module_t  ngx_http_core_module;
extern ngx_module_t  ngx_http_log_module;
extern ngx_module_t  ngx_http_upstream_module;
extern ngx_module_t  ngx_http_static_module;
extern ngx_module_t  ngx_http_autoindex_module;
extern ngx_module_t  ngx_http_index_module;
extern ngx_module_t  ngx_http_mirror_module;
extern ngx_module_t  ngx_http_try_files_module;
extern ngx_module_t  ngx_http_auth_basic_module;
extern ngx_module_t  ngx_http_access_module;
extern ngx_module_t  ngx_http_limit_conn_module;
extern ngx_module_t  ngx_http_limit_req_module;
extern ngx_module_t  ngx_http_geo_module;
extern ngx_module_t  ngx_http_map_module;
extern ngx_module_t  ngx_http_split_clients_module;
extern ngx_module_t  ngx_http_referer_module;
extern ngx_module_t  ngx_http_rewrite_module;
extern ngx_module_t  ngx_http_proxy_module;
extern ngx_module_t  ngx_http_fastcgi_module;
extern ngx_module_t  ngx_http_uwsgi_module;
extern ngx_module_t  ngx_http_scgi_module;
extern ngx_module_t  ngx_http_memcached_module;
extern ngx_module_t  ngx_http_empty_gif_module;
extern ngx_module_t  ngx_http_browser_module;
extern ngx_module_t  ngx_http_upstream_hash_module;
extern ngx_module_t  ngx_http_upstream_ip_hash_module;
extern ngx_module_t  ngx_http_upstream_least_conn_module;
extern ngx_module_t  ngx_http_upstream_random_module;
extern ngx_module_t  ngx_http_upstream_keepalive_module;
extern ngx_module_t  ngx_http_upstream_zone_module;
extern ngx_module_t  ngx_http_write_filter_module;
extern ngx_module_t  ngx_http_header_filter_module;
extern ngx_module_t  ngx_http_chunked_filter_module;
extern ngx_module_t  ngx_http_range_header_filter_module;
extern ngx_module_t  ngx_http_gzip_filter_module;
extern ngx_module_t  ngx_http_postpone_filter_module;
extern ngx_module_t  ngx_http_ssi_filter_module;
extern ngx_module_t  ngx_http_charset_filter_module;
extern ngx_module_t  ngx_http_userid_filter_module;
extern ngx_module_t  ngx_http_headers_filter_module;
extern ngx_module_t  ngx_http_copy_filter_module;
extern ngx_module_t  ngx_http_range_body_filter_module;
extern ngx_module_t  ngx_http_not_modified_filter_module;
```

最后形成`*ngx_modules[]`数组 .

#### make编译

编译完成以后 , 如果没有任何错误 , 这个时候可以看到生成了大量的中间文件 , 已经最终的运行的nginx二进制文件 , 可以在objs目录看到 .

```
./objs
├── Makefile
├── autoconf.err
├── nginx*
├── nginx.8
├── ngx_auto_config.h
├── ngx_auto_headers.h
├── ngx_modules.c
├── ngx_modules.o
└── src/
```

如果做Nginx版本升级 , 不能执行make install , 需要把nginx目标文件拷贝到安装目录中 . C语言在编译时生成的所有中间文件 , 都会放在objs/src目录下 . 如果使用了动态模块 , 编译会生成so动态文件 , 也会放在objs文件下 .  

