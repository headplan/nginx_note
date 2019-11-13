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

提供了两个perl脚本 , 和vim的配置 . 

```
contrib
├── README
├── geo2nginx.pl
├── unicode2nginx
│   ├── koi-utf
│   ├── unicode-to-nginx.pl
│   └── win-utf
└── vim
    ├── ftdetect
    │   └── nginx.vim
    ├── ftplugin
    │   └── nginx.vim
    ├── indent
    │   └── nginx.vim
    └── syntax
        └── nginx.vim
```





