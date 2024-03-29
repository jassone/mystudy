## nginx负载均衡

负载均衡，即Nginx的反向代理实现一个重要功能。

## 一、负载均衡算法

### 1、静态负载均衡算法

主要包括轮询算法、基于比率的加权轮询算法或者基于优先级的加权轮询算法。

### 2、动态负载均衡算法

主要包括基于任务量的最少连接优化算法、基于性能的最快响应优先算法、预测算法及动态性能分配算法等。

### 3、二者对比

静态负载均衡算法在一般网络环境下也能表现的比较好，动态负载均衡算法更加适用于复杂的网络环境。

* **轮询法(默认)**：将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。

* **加权轮询法(weight)**：不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此它们的抗压能力也不相同。给配置高、负载低的机器配置更高的权重，让其处理更多的请；而配置低、负载高的机器，给其分配较低的权重，降低其系统负载，并将请求顺序且按照权重分配到后端。**该算法能很好地处理分配高低负载服务器的分配问题**。

* **基于IP路由负载算法(ip_hash)**：根据获取客户端的IP地址，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是客服端要访问服务器的序号。采用源地址哈希法进行负载均衡，同一IP地址的客户端，当后端服务器列表不变时，它每次都会映射到同一台后端服务器进行访问。**可以解决session一致问题**。

* **url_hash**：与ip_hash类似，但是按照访问url的hash结果来分配请求，使得每个url定向到同一个后端服务器，**主要应用于后端服务器为缓存时的场景下。**

* fair法（非官方）：按照服务器响应时间的长短来进行分发的，**服务器响应时间越短的，优先分发**。需要安装第三方模块。

* 最少连接：**将请求分配到连接数最少的服务上**。

## 二、配置

### 1、upstream

todo

https://www.cnblogs.com/jxxiaocao/articles/16175562.html

