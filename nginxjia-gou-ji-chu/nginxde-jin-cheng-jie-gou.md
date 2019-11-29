# Nginx的进程结构

Nginx有两种进程结构 , **单进程结构**和**多进程结构** . 

**单进程结构**不适用于生产环境 , 只适用于开发和调试 . 生产环境中 , 必须保证nginx足够健壮 , 并且nginx可以利用多核的特性 , 单进程结构无法满足 . 所以nginx的默认配置中都是打开为多进程结构的nginx . 配置文件 : 

```
worker_processes auto;
或者
worker_processes 8; # CPU核数的2倍
```



