## main



- comment集合如果不存在，则会隐式创建
- mongo中的数字，默认情况下是double类型，如果要存整型，必须使用函数NumberInt(整型数字)，否则取出来就有问题了。
- 插入当前日期使用 new Date()
- 插入的数据没有指定 _id ，会自动生成主键值
- 如果某字段没值，可以赋值为null，或不写该字段。

 ## 相关命令

```sh
mongod -version // 查看版本

#启动
mongod --fork -dbpath /usr/local/mongodb/data --logpath /usr/local/mongodb/log/mongo.log --logappend 

## 连接mongodb
mongo 

## 数据库增删改查常用命令
db.study.find().pretty() #格式化结果

```



## 相关wiki



- 官网 https://www.mongodb.com/
- 官网中文 https://www.mongodb.org.cn/
- 中文文档 https://docs.mongoing.com/ 这里比较全
- 中文社区 https://mongoing.com/
- 教程 https://www.runoob.com/mongodb
- 极客时间课程 https://gitee.com/geektime-geekbang/geektime-mongodb-course