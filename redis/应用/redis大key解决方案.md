## redis大key解决方案

**bigkey是指key对应的value所占用的内存空间比较大。**

redis大Key一般达到多少，性能就下降了？**大概是>=10kb。** 

因为redis用的动态字符串(string类型)的库在**每次分配空间的时候会增加一倍的可用空闲空间**，存储大的字符时占空间大

- 字符串类型：体现在单个value过大
- 非字符串类型：比如哈希，列表，集合，有序集合，体现在元素过多。

## 一、场景
比如某个热门书籍的读者列表，用set存储。

产生的问题：
* 当读者用户大的时候，集合占用空间就大了。
* 而且热门书籍，访问量也大，导致redis qps下降。

## 二、危害

* **内存占用过大**。一旦内存吃紧，会频繁LRU清除，而客户端来读取时又会导致缓存穿透。
* **超时阻塞**：由于redis单线程特性，操作bigkey比较耗时，就意味着阻塞redis的可能性增大。
* **网络阻塞**：每次获取bigkey产生的网络流量较大。
* **qps突变**：大key相关的删除或者自动过期时，会出现qps突降或者突升的情况。
* **集群倾斜问题，负债不均衡**。导致某个节点存储和qps压力大(原因同上)。
    集群中key按照一定规则(比如hash算法)被分发到不同节点上，如果某个key过大，导致该节点内存占用大。
## 三、优化思路
### 1、结合业务，存储这么大量的数据是否有意义/必要
### 2、去掉不需要的存储字段
### 3、开启redis6的客户端缓存(重要)
常规做法是用本地缓存(比如Ehcache)。但是为了保持数据一致性，如果redis数据发送变化，一般需要通过双写刷新本地缓存。

而redis6客户端缓存，它自己就能自动更新缓存，简化操作。

## 四、查找大key工具
思路都是分析rdb文件，查找大key。
### 1、rdb_bigkeys--推荐
go写的一个工具，3G的rdb文件，3分钟就能执行完。

### 2、redis-rdb-tools工具。
redis实例上执行bgsave，然后对dump出来的rdb文件进行分析，找到其中的大KEY。

### 3、redis-cli --bigkeys命令。
Redis-cli --bigkeys是redis-cli自带的一个命令。它对整个redis进行扫描，寻找较大的key，并打印统计结果。

可以找到某个实例5种数据类型(String、hash、list、set、zset)的最大key。

### 4、memory usage命令(Redis >=4.0)

### 5、大key删除方法(Redis >=4.0)
lazyfree机制：Lazyfree的原理是在删除的时候只进行**逻辑删除**，把key释放操作放在bio(Background I/O)单独的子线程处理中。

### 4、自定义的扫描脚本
以Python脚本居多，方法与redis-cli --bigkeys类似。

