## clickhouse

ClickHouse不单单是一个数据库， 它是一个数据库管理系统。因为它允许在运行时创建表和数据库、加载数据和运行查询，而无需重新配置或重启服务。

由于ROLAP和MOLAP的缺点，clickhouse转而使用搜索引擎来实现OLAP(ElasticSearch也是这种方案，支持实时查询)。

用于联机分析OLAP的**列式数据库**，支持sql实时查下的大型数据管理系统。

## 一、特点

### 1、OLAP

其更关注海量数据的计算分析，数据吞吐，查下速度，计算性能等指标，对数据排列的修改变更则不擅长。所以其通常用来构建后端的实时数仓或离线数仓。

### 2、列式存储

不同列的数据进行单独存储，一列为一个文件。 好处是数据分析查询会很快。

## 二、应用场景

### 1、试用场景

- 绝大多数请求都是读请求，对数据的修改比较少或者几乎没有。
- 数据量很大，包含行和列都很大。
- 数据通常以大的批次进行整体更新，而不是单行更新。这需要很高的数据吞吐量。
- 对事物的要求不是必须的，通常只是数据最终一致性。

### 2、不适用的场景

- 不支持事务
- 不擅长根据主键按行力度进行查询(虽然支持)，故不应把ch当做key-value数据库使用。
- 不擅长按行删除数据(虽然支持) 

## 三、其他

### 缺点

高性能的背后，是计算机资源的大量消耗，特别是对cpu和内存占用率非常高。





官网文档 https://clickhouse.com/docs/zh/

b站教程 https://www.bilibili.com/video/BV1xg411w7AP/
