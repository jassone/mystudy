## pipeline



```go
pipeline = [
  {'$project':{'minu': 1, '_id': 0, 't': 1, 'r': 1}},
  {'$unwind': '$minu'},
  {'$match': {'minu': {'$ne': None}}},
  {'$project': {'r': 1, 'trans_count': '$minu.k', 'ts': {'$add': ['$t', '$minu.t']}}},
  {'$match': {'ts': {'$gte': 1457059140, '$lt': 1457060040}}},
  {'$project': {'trans_count': 1, 'id': 0, 'r': 1}},
  {'$group': {'trans_count': {'$sum': '$trans_count'}, 'id': '$r'}},
  {'$project': {'trans_count': 1, 'id': 0, 'ret_code': '$id'}},
  {'$sort': {'trans_count': -1, 'ret_code': 1}}
]
```

 mongodb会顺序执行以下内容：
①获取minu, t, r 字段的内容, 去掉_id字段
②分解minu的数据
③去掉minu字段为null的数据
④将保留r字段, minu.k重命名为trans_count, ts为t和minu.t的和
⑤获取ts在[1457059140, 1457060040]之间的数据
⑥获取trans_count, r 字段
⑦按照r字段的值分组，同一个r值得trans_count相加（id的值是r字段的内容）
⑧保留trans_count字段,_id重命名为ret_code
⑨根据trans_count降序，ret_code升序排列



```go
##获取发布时间为2020.03.21，且次数为3次
##对 price 字段进行分组并统计次数
pipeline = [
  {"$match":{"$and":[{"pub_time":"2020.03.21"},{"time":3}]}},
  {"$group":{"_id":$price,"counts":{"$sum":1}}},
  {"$sort":{"counts":-1}},
  {"$limit":3},
]

##获取发布时间为2020.03.21，且区域包含上海
#根据 cates 字段进行分组然后分隔，并统计次数
pipeline = [
  {"$match":{"$and":[{"pub_time":"2020.03.21"},{"area":{"$all":['上海']}}]}},
  {"$group":{"_id":{"$slice":["$cates",2,1]},"counts":{"$sum":1}}},
  {"$limit":3},
]

##获取发布时间为2020.03.21，且区域包含上海，且look字段不包含 '-'
#根据 look 字段进行分组，且 求每组 price 字段的平均值
pipeline = [
  {"$match":{"$and":[{"pub_time":"2020.03.21"},{"area":{"$all":['上海']}},{'look':{'$nin':['-']}}]}},
  {"$group":{"_id":"$look","avg_price":{"$avg":$price}}},
  {"$sort":{'avg_price':-1}},
]

##直接分组
db.colliction.group({ 
  "key": {'storeName': 1},      //storeName 存在
  cond: {"$and":[{ "createTime": { $gte:  ISODate("2020-02-10T00:00:00Z") } },{ "createTime": { $lte:  ISODate("2020-02-20T23:59:59Z") } }]},   //取出指定时间段
  reduce: function(obj,storeName) {storeName.count++},  //对storeName 做自增运算
  initial: { count:0}  //初始值为0
})
```

