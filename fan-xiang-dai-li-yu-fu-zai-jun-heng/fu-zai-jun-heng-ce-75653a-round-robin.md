# 负载均衡策略:round-robin

Nginx负责与上游服务交互的模块统称为upstream模块 , 不管是stream , upstream还是http\_upstream . 

在upstream模块中 , 除了指定上游服务器以外, 还提供了一个最基本的负载均衡算法 , round-robin算法 . 





