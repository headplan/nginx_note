# 反向代理与负载均衡原理

#### 负载均衡

![](/assets/fuzaijunh.png)

上面的场景就是一种负载均衡场景 , Nginx提高了应用服务集群的可用性 . 包括容灾 , 扩容等 .

#### Nginx在AKF扩展立方体上的应用

> **AKF扩展立方体\(Scalability Cube\)** , 是《架构即未来》一书中提出的可扩展模型 , 这个立方体有三个轴线 , 每个轴线描述扩展性的一个维度 , 他们分别是产品、流程和团队 :
>
> * X轴 —— 代表无差别的克隆服务和数据 , 工作可以很均匀的分散在不同的服务实例上 ; 
> * Y轴 —— 关注应用中职责的划分 , 比如数据类型 , 交易执行类型的划分 ; 
> * Z轴 —— 关注服务和数据的优先级划分 , 如分地域划分 .

![](/assets/akflifangti.png)

#### 三个维度扩展的对比

通过这三个维度上的扩展 , 可以快速提高产品的扩展能力 , 适应不同场景下产品的快速增长 . 不同维度上的扩展 , 有着不同的优缺点 :

**1.X轴扩展**

X轴扩展 , 服务是无状态的 , 无论起多少个服务 , 都是同等的为用户请求服务 , 扩容的成本也是最低的 , 也就是我们通常说的水平扩展 . Nginx的**Round-Robin**或者**least-connected**算法都是标准的基于水平扩展的负载均衡算法\(其他的如哈希算法也可以基于水平扩展理论去执行\) . 当然水平扩展不能解决所有问题 , 特别是数据量的问题 , 当单台应用上的数据已经非常大的时候 , 无论扩展多少台服务 , 每一台服务的数据仍然非常大 .

* 优点 : 成本最低 , 实施简单 ;
* 缺点 : 受指令集多少和数据集大小的约束 . 当单个产品或应用过大时 , 服务响应变慢 , 无法通过X轴的水平扩展提高速度 ;
* 场景 : 发展初期 , 业务复杂度低 , 需要增加系统容量 .

**2.Y轴扩展**

Y轴扩展从功能上进行拆分 , 例如 , 原先有一台应用服务 , 拆分为两台 , 他们分别处理不同的API , 也就是不同的URL , 这时候完全可以用Nginx中的location去配置 . 比如一些location我们用proxy\_pass代理到一些上游服务 , 另外一些URL代理到另外的集群的URL服务中 . 就实现了Y轴的扩展 . Y轴的扩展通常需要大量的重构 , 修改代码 , 所以说成本是比较高的 . 但是它可以解决数据的上升问题 .

* 优点 : 可以解决指令集和数据集的约束 , 解决代码复杂度问题 , 可以实现隔离故障 , 可以提高响应时间 , 可以使团队聚焦更利于团队成长 ;
* 缺点 : 成本相对较高 ;
* 场景 : 业务复杂 , 数据量大 , 代码耦合度高 , 团队规模大 .

**3.Z轴扩展**

Z轴扩展是基于用户的信息进行扩展 , 比如说 , 可以基于用户的IP地址 , 我没发现有些IP地址比较靠近某一个CDN中心 , 就把这样的请求引流到那个CDN上 . 为了分离减小数据容量 , 也可以根据用户名 , username等 , 某些固定的用户 , 引流到负载均衡到某一个固定的集群上 , 这也是基于Z轴 . Nginx提供了很多基于Hash的算法 , 将用户IP地址或者其他信息映射到某个特定的服务或集群 .

* 优点 : 能解决数据集的约束 , 降低故障风险 , 实现渐进交付 , 可以带来最大的扩展性 ;
* 缺点 : 成本最昂贵 , 且不一定能解决指令集的问题 ;
* 场景 : 用户指数级快速增长 .

**实际上XYZ通常会组合应用 , Nginx也支持这些操作 . **

#### 如何将理论付诸实践

**1.为扩展分割应用**

* X轴 : 从单体系统或服务 , 水平克隆出许多系统 , 通过负载均衡平均分配请求 ; 
* Y轴 : 面向服务分割 , 基于功能或者服务分割 , 例如电商网站可以将登陆、搜索、下单等服务进行Y轴的拆分 , 每一组服务再进行X轴的扩展 ; 
* Z轴 : 面向查找分割 , 基于用户、请求或者数据分割 , 例如可以将不同产品的SKU分到不同的搜索服务 , 可以将用户哈希到不同的服务等 . 

**2.为扩展分割数据库**

* X轴 : 从单库 , 水平克隆为多个库上读 , 一个库写 , 通过数据库的自我复制实现 , 要允许一定的读写时延 ; 
* Y轴 : 根据不同的信息类型 , 分割为不同的数据库 , 即分库 , 例如产品库 , 用户库等 ; 
* Z轴 : 按照一定算法 , 进行分片 , 例如 , 将搜索按照MapReduce的原理进行分片 , 把SKU的数据按照不同的哈希值进行分片存储 , 每个分片再进行X轴冗余 . 

**3.为扩展而缓存**

在理想情况下 , 处理大流量最好的方法是通过高速缓存来避免处理它 . 从架构层面看 , 我们能控制的主要有以下三个层次的缓存 :

**对象缓存** : 对象缓存用来存储应用的对象以供重复使用 , 一般在系统内部 , 通过使用应用缓存可以帮助数据库和应用层卸载负载 .

**应用缓存** : 应用缓存包括代理缓存和反向代理缓存 , 一个在用户端 , 一个在服务端 , 目标是提高性能或减少资源的使用量 .

**内容交付网络缓存** : CDN的总原则是将内容推送到尽可能接近用户终端的地方 , 通过不同地区使用不同ISP的网关缓存 , 达到更快的响应时间和对源服务的更少请求 .

**4.位扩展而异步**

**同步改异步** : 同步调用 , 由于调用间的同步依赖关系 , 有可能会导致雪崩效应 , 出现一系列的连锁故障 , 进而导致整个系统出现问题 , 所以在进行系统设计时 , 要尽可能的考虑异步调用方式 , 邮件系统就是一个非常好的异步调用例子 .

**应用无状态 : **当进行AKF扩展立方体的任何一个轴上的扩展时 , 都要首先解决应用的状态问题 , 即会话的管理 , 可以通过避免、集中和分散的方式进行解决 .

---

AKF扩展立方体是一套通用的扩展性理论 , 它不仅可以应用到系统的架构扩展上 , 也可以应用到人员的组织架构扩展上甚至其他相关的工业领域 .

当然并不是所有公司都需要同时在XYZ三个方向上进行扩展 , 并且每个方向上的扩展都有它的利弊 , 我们不可避免的要进行适当的权衡 . 最重要的 , 我们应当首先理解这套理论背后所体现出来的扩展哲学 .

---

#### 反向代理

Nginx支持多种协议的反向代理 . 反向代理主要分为两类 , 4层和7层 .

4层的反向代理 , Nginx的stream模块支持 . 进来是UDP流量 , Nginx转发到上游的还是UDP流量 , TCP流量也是一样的 . 这两种协议的业务特性相对较少 , Nginx并不能做很多工作 , 所以相对比较简单 .

应用层 , 也就是7层的反向代理的时候就不太一样了 , HTTP协议中含有大量的业务相关的信息 , 当客户端发来http请求以后 , 可以根据http的header , method等等参数的信息将其转为不同的协议 .

![](/assets/fanxiangdaili.png)

---

#### 反向代理预缓存

![](/assets/fandaiyuhuancun.png)

