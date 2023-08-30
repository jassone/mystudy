## crud

### 1、更新操作

$push 增加一个对象到数组底部
$pushAll 增加多个对象到数组底部
$pop 从数组底部删除一个对象
$pull 如果匹配指定的值，从数组中删除相应的对象
$pullAll  如果匹配任意的值，从数组中删除相应的对象
$addToSet 如果不存在则增加一个值到数组

### 2、$bucket 操作

Todo

### 3、$Facet

比如一次查询出总数和分页数据

```go
pipeline := mongo.Pipeline{
   {{"$facet", bson.M{
      "total_size": []bson.M{{"$match": queryFilter}, {"$group": bson.M{"_id": "$plat_aid", "count": bson.M{"$sum": 1}}}, {"$project": bson.M{"_id": 0}}},
      "data":       []bson.M{{"$match": queryFilter}, {"$skip": (pageNow - 1) * pageSize}, {"$limit": pageSize}},
   }}},
}
```

https://devpress.csdn.net/mongodb/62f98dc07e6682346618d68b.html

### 4、$elemMatch 查询

就是说$elemMatch是用来查询数组字段的，如果数组字段中有至少1个元素匹配查询规则，则查询出来这个数组

```
// 官方标准语法
{ <field>: { $elemMatch: { <query1>, <query2>, ... } } }
// field是数组字段名，在$elemMatch中传入查询条件，可多个，用逗号隔开。

// 语句说明，查找 results数组中存在大于等于80且小于85的元素的文档。（只要元素中有一个匹配，那么这个元素所在的数组的文档就会返回）
db.scores.find(
   { results: { $elemMatch: { $gte: 80, $lt: 85 } } }
)
```

### 5、联表查询

 lookup使用时，a表字段可以是一个数组，这样查询出来的as是一个数组

 引用的限制：
 lookup只支持left join
 lookup的关联表(form)不能是分片表

