## MySql Sharding

## 一、mysql sharding和mysql cluster对比
mysql cluster 只是一个数据库集群，优势只是扩展了数据库的并行处理能力，而且使用成本和维护成本比较高，**目前在生产上使用较少了**。

而mysql sharding是一个成熟的方案，不仅可以提升数据库的并行处理能力，还能解决因为单表数据量大所产生的检索瓶颈。**是一种分布式模式，是当下互联网的首选**。

## 二、sharding中间件
### 1、分库分表时，需要涉及两个问题：
* 必须明确定义sql语句中的shard key(路由条件)，因为路由维度决定了数据的落盘位置。
* 如何根据所定义的shard key进行路由访问，这需要定义一套特定的路由算法和规则。

### 2、常见中间件对比
![DCF-A0A636615B9B.png](https://pic.imgdb.cn/item/62b09b7009475431297fa135.png)

其中shark的文档比较丰富，可以考虑。

## 相关wiki
* MySql Sharding分表、分库、分片和分区知识讲解 https://blog.51cto.com/u_1472521/3714520

