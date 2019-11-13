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



