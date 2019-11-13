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
├── CHANGES - 每个Nginx版本提供的特性feature和bugfix
├── CHANGES.ru - 俄语版
├── LICENSE - 
├── README
├── auto/
├── conf/
├── configure*
├── contrib/
├── html/
├── man/
└── src/
```



```
├── auto/
│   ├── cc/
│   ├── define
│   ├── endianness
│   ├── feature
│   ├── have
│   ├── have_headers
│   ├── headers
│   ├── include
│   ├── init
│   ├── install
│   ├── lib/
│   ├── make
│   ├── module
│   ├── modules
│   ├── nohave
│   ├── options
│   ├── os/
│   ├── sources
│   ├── stubs
│   ├── summary
│   ├── threads
│   ├── types/
│   └── unix
```



