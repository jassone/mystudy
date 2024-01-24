## 普通sql

## 一、更新/插入

### 1、insert ignore into
表示如果中已经存在相同的记录，则忽略当前新数据。

当插入数据时，如出现错误时，如重复数据，将不返回错误，只以警告形式返回。所以使用ignore请确保语句本身没有问题，否则也会被忽略掉。例如：

```sql
INSERT IGNORE INTO books (name) VALUES ('MySQL Manual')
```

### 2、on duplicate key update：
当primary或者unique重复时，则执行update语句，如update后为无用语句，如id=id，则同1功能相同，但错误不会被忽略掉。例如，为了实现name重复的数据插入不报错，可使用一下语句：

```sql
INSERT INTO books (name) VALUES ('MySQL Manual') ON duplicate KEY UPDATE id = id
```

### 3、update case

Case具有两种格式。简单Case函数和Case搜索函数。

```sql
---简单Case函数
CASE sex
WHEN '1' THEN '男'
WHEN '2' THEN '女'
ELSE '其他' END

--Case搜索函数
CASE WHEN sex = '1' THEN '男'
WHEN sex = '2' THEN '女'
ELSE '其他' END

UPDATE test 
SET count =
CASE
		WHEN (count - 11)> 0 THEN (count - 11) 
		ELSE count 
	END 
WHERE
	id = 3
```

和使用普通where条件判断对比，使用case时都会去执行，但使用where时如果不符合条件，就不去执行了。

### 4、update exists

```sql
UPDATE parent_table pt SET pt.data = 'new data' WHERE EXISTS (  SELECT 1 FROM child_table ct WHERE ct.parent_id = pt.id );
```

### 5、复制表数据

```sql
INSERT INTO table_name (column1, column2, ...) SELECT column1, 'new_value', ... FROM Temp_table WHERE condition;
```

## 二、查询

### 1、limit和offset

* limit y 分句表示: 读取 y 条数据
* limit x, y 分句表示: 跳过 x 条数据，读取 y 条数据
* limit y offset x 分句表示: 跳过 x 条数据，读取 y 条数据

### 2、where和having

where子句：是在分组之前使用，表示从所有数据中筛选出部分数据，以完成分组的要求，在where子句中不允许使用统计函数，没有group by子句也可以使用。
                                        
having字句：是在分组之后使用的，**表示对分组统计后的数据执行再次过滤**，可以使用统计函数，有group  by子句之后才可以出现having子句。

二者都可以用

```sql
select goods_price,goods_name from sw_goods where goods_price > 100
    
 select goods_price,goods_name from sw_goods having goods_price > 100
```

只能用where

```sql
select goods_name from sw_goods where goods_price > 100
```

只能用having

```sql
select goods_category_id , avg(goods_price) as ag from sw_goods group by goods_category having ag > 1000
```

### 3、exists

```sql
select * from emp where EXISTS (select id from dept where dept.id=emp.dep_id);
```

exists后面的子查询不返回任何实际数据，只返回真或假，当返回真时 where条件成立，该条记录保留。

### 4、强制使用索引

要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

```sql
select * from msg force index(src_des) where src = '13' 
```

### 5、any/some

只要存在一个t2.m2<t1.m1的情况，整个布尔表达式就返回true

```sql
select * from t1 where m1 > any(select m2 from t2)
```

### 6、all

要求t2.m2 都小于 t1.m1时，整个布尔表达式才返回true

```sql
select * from t1 where m1 > all(select m2 from t2)
```
