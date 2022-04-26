## MongoDB

todo

### 优点
* 反范式设计，结构灵活，方便管理，存储全量快照
* 天然支持json,扩展性强
* 还支持json的schema查询

https://www.jianshu.com/p/69ffcd3c254b

### 适用场景
* 特别适合分布式noSql，适合海量数据存储（分片支持百亿数据）
* 医疗处方，因为字段类型非常不确定，直接存json
* 表记录变更历史，直接将当前所有数据(快照)打包成json

