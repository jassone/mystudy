## main

### 1、深度分页如何优化

- 给查询字段建立索引

#### 方案1 - 利用子查询 或者 二次查询

- 通过子查询，利用索引覆盖，查找第n行页的起始数据，快速定位 --- 比如  limit 10000,1
- 主sql limit 10条数据，查询需要的相关字段 --- 减少大量回表查询

#### 方案2 - 基于上下页

- 利用上一页最后一条记录的值，来作为下一页查询的起始值，可以减少一次子查询
- 缺点：只能连续分页

#### 方案3 - 换其他数据库

#### 方案四 - 限制查询页码



### 相关文档

- https://www.bilibili.com/video/BV1ts4y1u7EK/?spm_id_from=333.1007.tianma.2-2-4.click&vd_source=c4945c7a8bb113be5f156e95d4726dfe

- 必看  海量数据大页码MySQL查询该如何优化？https://blog.csdn.net/panjianlongWUHAN/article/details/122670773



