## 表引擎-MergeTree

MergeTree是clickhouse非常有特色的一个特性，非常重要。它决定了一个表的所有数据属性。

MergeTree引擎底层使用一种类似**LSM树**的结构来保存数据。

## 一、相关参数

### 1、partition by-分区键

其作用是为了降低数据扫描范围，不用进行全表扫描，优化查询速度。如果不填默认即一个分区(一个文件)。 

分区键可以是组合方式：比如 **partition by (length(code), EventDate)**。

### 2、ORDER BY — 排序键

唯一的一个必填项，指定分区内的数据按照哪些字段进行有序保存。

在海量数据场景下，实现快速检索，去重，汇总等计算都离不开数据有序的支持。**注意：clickhouse的数据是分区内局部有序的(因为clickhouse对于数据的处理就是以分区作为最小维度的)**

### 3、primary key - 主键

- 相当于对排序键再进行一次索
- clickhouse不要求主键唯一。
- 常驻内存。

默认情况下主键跟排序键相同。大部分情况下不需要再专门指定一个 `PRIMARY KEY` 子句。如果需要设置一个不一样的主键，则必须是order by的前缀字段，比如排序字段为(id,sku_id),那么主键只能是id或者(id,sku_id)。

clickhouse有个重要的参数**index granularity(索引粒度)**，默认为8192,是指在**稀疏索引**中两个相邻索引对应的数据的间隔。通常不需要修改，如果一个分区中重复的列特别多，可以考虑改小。

![20221226140050.jpg](https://pic.imgdb.cn/item/63a938a708b683016384de5d.jpg)

##### a) 二级索引

```sql
## 在total_amount建立一个二级索引，粒度为5
create table t_stock2(
	id UInt32,
	sku_id String,
	total_amount Decimal(16,2),
	create_time DateTime,
	index secondIndex total_amount TYPE minmax GRANULARITY 5
) engine = MergeTree()
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id,sku_id);
```

相当于在一级索引上再加一层索引，加快检索速度。？？？

### 4、TTL - 数据存活时间

用来指定行存储的持续时间。

##### a) 列级TTL

```sql
create table t_stock_ttl1(
  id UInt32,
	d DateTime,
	a Int TTL d + interval 1 month,
	b Int TTL d + interval 1 MONTH
) engine = MergeTree()
partition by toYYYYMMDD(d)
primary key (id)
order by id
```

如果过期了，clickhouse会把他们替换成数据类型的默认值。如果某个数据块中列的所有值都过期了，clickhouse就会直接删除数据块中这一列。

列式TTL不能用于主键。

##### b）表级TTL

除了设置一个过期表达式之外，还可以配置一个数据移除规则。

```sql
TTL expr 
    [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
    [where condition]
    [group by key_expr [set v1=aggr_func(v1) [,...]]]
      
create table t_stock_ttl1(
    a UInt32,
	d DateTime
) engine = MergeTree()
partition by toYYYYMMDD(d)
order by d
TTL d + interval 1 month [DELETE],   # DELETE不写就是默认删除
    d + interval 1 week to volume 'aaa',
    d + interval 2 week to disk 'bbb';
```

- Delete : 删除过期的行，默认

- TO DISK 'aaa' : 将数据块移动到磁盘'aaa'

- TO VOLUME 'bbb': 将数据块移动到卷'bbb'

- GROUP BY :集合过期的行

  后面的where可以指定哪些过期的行为会被删除或聚合（不适用数据移动）

### 5、SAMPLE BY - 数据抽样

用于大数据分析，不需要全量分析，可以极大提升数据分析性能。

```sql
## 建表时加上
SAMPLE BY intHash32(UserID)
## 查询
select title,count(*) as pageview
from hits_v1
sample 0.1 #代表10%的数据，也可以是具体的条数
where CounterID = 57
group by title limit 1000
```

## 二、合并

插入时先写入到一个新的**临时分区**中，然后再按照一定时机(大概10-15分组)刷新到磁盘(涉及到合并)，读取时将磁盘和新分区中的数据再合并(涉及到合并)后返回给客户端，所以插入会很快，而读取会较慢。

合并后会产生一个新的分区，而之前的两个老的分区就可以废弃了(clickhouse后期会有个大合并会删除无用分区)。

```sh
optimize table t_stock final; 手动操作合并
```

