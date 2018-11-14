# Nginx配置

### 研究Nginx前的准备

**Nginx是什么和一些概念**

Web服务器:基于REST架构风格,以统一资源描述符或统一资源定位符作为沟通依据,通过HTTP为浏览器等客户端程序提供各种网络服务.

* Apache:发展时间长,重量级,但不支持高并发.但是稳定,开源跨平台.
* Nginx:轻量级,高性能Web服务器,国内公司用的比较多

  * Nginx使用基于事件驱动的架构能够并发处理百万级的TCP连接.

* Lighttpd:轻量级,高性能Web服务器,欧美开发者较青睐

* Tomcat:面向Java,重量级Web服务器

* Jetty:面向Java,重量级Web服务器

* IIS:只能在Windows操作系统上运行

Nginx是一个跨平台的Web服务器,可以运行在不同的操作系统上,并且还可以使用当前操作系统的特有API来提高自己的性能.  
例如,使用Linux上的epoll来处理大并发网络连接.又如,对于Linux,Nginx支持其独有的sendfile系统调用,这个系统调用可以高效的把硬盘中的数据发送到网络上,而不需要先把硬盘数据复制到用户内存上再发送.

**为什么选择Nginx**

* 更快
* 高扩展性
* 高可靠性
* 低内存消耗
* 单机支持10万以上的并发连接
* 热部署
* 最自由的BSD许可协议

### 准备工作

**Linux操作系统**

内核需要在2.6以上,只有2.6以上才支持epoll.

使用`uname -a`查看Linux内核的版本.

**使用Nginx的必备软件**

yum install -y gcc : 编译C语言程序,Nginx不会直接提供二进制可执行程序.

yum install -y gcc-c++ : 编写Nginx HTTP模块时会用到.

yum install -y pcre pcre-devel : pcre是正则表达式库,pcre-devel是二次开发时用的开发库\(nginx.conf配置文件里会用正则\)

yum install -y zlib zlib-devel : 对HTTP包的内容做gzip格式的压缩\(nginx.conf配置文件里可以开启gzip on\)

yum install -y openssl openssl-devel : 应用SSL协议传输HTTP得安装,要用MD5,SHA1等散列函数也得安装.

**磁盘目录**

* 源代码存放目录 - 该目录用于存放源码和第三方或我们自己写的模块源码文件

* 编译阶段产生的中间文件存放目录 - 存放configure命令执行后所生成的源文件及目录,还有make命令执行后生成的目标文件和最终连接成功的二进制文件.默认情况下configure命令会将目录命名为objs,放在源代码目录下.

* 部署目录 - 该目录存放服务运行期间所需要的二进制文件,配置文件等,默认在\/usr\/local\/nginx

* 日志文件存放目录 - 日志文件通常会比较大,研究底层架构时候,会打开debug级别的日志,非常详细,日志文件会很大.

**Linux内核参数的优化**

默认的Linux系统考虑的是最普通的通用场景.需要修改之后,使得Nginx可以拥有更高的性能.

在内核优化时,Nginx作为静态Web内容服务器,反向代理服务器或是提供图片缩略图功能\(实时压缩图片\)的服务器时,内核参数的调整都是不同的,有针对性的.下面是最通用的使Nginx支持更多并发请求的TCP网络参数做简单说明.

修改`/etc/sysctl.conf`来更改内核参数,然后使用`sysctl -p`命令生效,最常用的配置 :

fs.file-max = 999999 \#

**Nginx源码下载**

下载地址:http://nginx.org/en/download.html

tar -zxvf nginx-xxxx.tar.gz

**编译安装Nginx**

./configure - 检测操作系统内核和已经安装的软件,参数的解析,中间目录的生成以及根据各种参数生成一些C源码文件,Makefile文件等

make - 根据上一步生成的Makefile文件编译Nginx工程,并生成目标文件和最终的二进制文件.

make install - 根据第一步执行时的参数将Nginx部署到指定的安装目录,包括目录的建立和二进制文件,配置文件的复制

最后可以make clean和make distclean清楚前面生成的东西.

**configure详解**

**Nginx的命令行控制**

最简单的`whereis nginx`命令,查看nginx的安装路径,执行文件\(二进制程序\)路径,配置文件路径和模块路径.

`echo $PATH`查看全局变量,如果设置了,就可以直接使用ngxin执行二进制程序.

1.默认方式启动  
直接执行nginx文件.这时会执行configure设置的--conf-path=PATH指定的nginx.conf配置文件.

2.自定义指定配置文件启动

nginx -c /tmp/nginx.conf

启动时加载指定的配置文件.

3.另行指定安装目录的启动方式

nginx -p /usr/local/nginx/

