# Nginx的请求处理流程

![](/assets/nginxqingqiuchuliliucheng.png)

Web , Email , TCP大致有三种流量进入 . Nginx中有三个大的状态机 : 

* 传输层状态机 : 处理TCP , UDP的四层的传输层状态机 . 
* HTTP状态机 : 处理应用层的HTTP状态机 . 
* Mail状态机 : 处理邮件的Mail状态机 . 


