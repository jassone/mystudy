

### $Facet

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



