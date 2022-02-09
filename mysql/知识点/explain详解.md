## EXPLAIN详解
EXPLAIN 语句来帮助我们查看某个查询语句的具体执行计划，从而可以有针对性的提升我们查询语句的性能。

## EXPLAIN输出字段
![20220123115133.jpg](https://pic.imgdb.cn/item/61ecd0ce2ab3f51d91db09f1.jpg)

### 1、id
SQL执行的成功的标识，SQL从大到小的执行。

* id相同时，执行顺序由上至下，内存会认为三个表，乘积小的先执行
* 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
 
### 2、select_type
表示当前的小查询在整个大查询中扮演了一个什么角色，有如下一些值：

##### 2-1、SIMPLE
最简单的select(不使用UNION或子查询等)  

##### 2-2、PRIMARY
对于包含 UNION 、 UNION ALL 或者子查询的大查询来说，它是由几个小查询组成的，其中最左边的那个查询或者最外层的select，类型就是PRIMARY。

##### 2-3、UNION
对于包含 UNION 或者 UNION ALL 的大查询来说，它是由几个小查询组成的，其中除了最左边的那个小查询以外，其余的小查询的 select_type 值就是 UNION 。

##### 2-4、DEPENDENT UNION
在包含 UNION 或者 UNION ALL 的大查询中，如果各个小查询都依赖于外层查询的话，那除了最左边的那个小查询之外，其余的小查询的select_type 的值就是 DEPENDENT UNION 。

```sql
EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2 WHERE key1 = 'a'
UNION SELECT key1 FROM s1 WHERE key1 = 'b');
```  
![20220123155007.jpg](https://pic.imgdb.cn/item/61ed08b72ab3f51d9111d04d.jpg) 

SELECT key1 FROM s1 WHERE key1 = 'b' 这个查询的。select_type 就是 DEPENDENT UNION

##### 2-5、SUBQUERY
非最外层的select，在子查询中第一个select，

```sql
EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2 where id=1) ;
```

##### 2-6、DEPENDENT SUBQUERY
子查询中的第一个select，取决于外面的查询。

``` sql
EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2 WHERE s1.key2 = s
2.key2);
```

##### 2-7、DERIVED
派生表的select(from子句的子查询)

```sql
EXPLAIN SELECT * FROM (SELECT key1, count(*) as c FROM s1 GROUP BY key1) AS derived_s1 where c > 1;
```

### 3、table
代表表名。有时不是真实的表名,看到的是<derivedX>(数字X是第几步执行的结果)

### 4、 type(非常重要)
使用了哪种类别，是否使用索引.

const、eq_reg、ref、range、index、ALL（**从左到右，性能从好到差**）

##### 4-1、const、system
根据主键或者唯一二级索引列与常数进行等值匹配时，对单表的访问方法就是 const。

system是const类型的特例，当查询的表只有一行的情况下，使用system。

##### 4-2、eq_ref
在连接查询时，如果**被驱动表是通过主键或者唯一二级索引列等值匹配的方式进行访问**的（如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较），则对该被驱动表的访问方法就是eq_ref。

##### 4-3、ref
当通过普通的二级索引列与常量进行等值匹配时来查询某个表，那么对该表的访问方法就可能是 ref。

```sql
EXPLAIN SELECT * FROM s1 WHERE key1 = 'a';
```

##### 4-4、ref_or_null
当对普通二级索引进行等值匹配查询，该索引列的值也可以是 NULL 值时，那么对该表的访问方法就可能是ref_or_null。

##### 4-5、fulltext
全文索引

##### 4-6、index_merge
一般情况下对于某个表的查询只能使用到一个索引，但我们唠叨单表访问方法时特意强调了在某些场景下可以使用 Intersection 、 Union 、 Sort-Union 这三种索引合并的方式来执行查询，就可能是index_merge。

##### 4-7、unique_subquery

##### 4-8、index_subquery

##### 4-9、range
如果使用索引获取某些 范围区间 的记录，那么就可能使用到 range 访问方法。

##### 4-10、index
全索引扫描。index与ALL区别为index类型只遍历索引树。

##### 4-11、ALL
MySQL进行全表扫描。
 
### 5、possible_keys
列表示在某个查询语句中，对某个表执行单表查询时可能用到的索引有哪些。

### 6、key(非常重要)
表示实际用到的索引有哪些。

### 7、key_len
key_len列显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好。

### 8、ref
当使用索引列等值匹配的条件去执行查询时，也就是在访问方法是 const 、 eq_ref 、 ref 、 ref_or_null 、unique_subquery 、 index_subquery 其中之一时， ref 列展示的就是与索引列作等值匹配的东东是个啥。


### 9、rows
rows列显示MySQL认为它执行查询时必须检查的行数。

### 10、filtered
![20220123162601.jpg](https://pic.imgdb.cn/item/61ed11262ab3f51d911a335a.jpg)

从执行计划的 key 列中可以看出来，该查询使用 idx_key1 索引来执行查询，从 rows 列可以看出满足 key1 >'z' 的记录有 266 条。执行计划的 filtered 列就代表查询优化器预测在这 266 条记录中，有多少条记录满足其余的搜索条件，也就是 common_field = 'a' 这个条件的百分比。此处 filtered 列的值是 10.00 ，说明查询优化器预测在 266 条记录中有 10.00% 的记录满足 common_field = 'a' 这个条件。

### 11、Extra
用来说明一些额外信息的，我们可以通过这些额外信息来更准确的理解 MySQL 到底将如何执行给定的查询语句。

##### 11-1、No tables used
当查询语句的没有 FROM 子句时将会提示该额外信息，比如：
```sql
EXPLAIN SELECT 1;
```

##### 11-2、Impossible WHERE
查询语句的 WHERE 子句永远为 FALSE 时将会提示该额外信息，比方说：
```sql
EXPLAIN SELECT * FROM s1 WHERE 1 != 1;
```

##### 11-3、No matching min/max row
当查询列表处有 MIN 或者 MAX 聚集函数，但是并没有符合 WHERE 子句中的搜索条件的记录时，将会提示该额外信息，比方说：
```sql
EXPLAIN SELECT MIN(key1) FROM s1 WHERE key1 = 'abcdefg';
```

##### 11-4、Using index
当我们的查询列表以及搜索条件中只包含属于某个索引的列，也就是在可以使用索引覆盖的情况下，在Extra 列将会提示该额外信息。比方说下边这个查询中只需要用到 idx_key1 而不需要回表操作：
```sql
EXPLAIN SELECT key1 FROM s1 WHERE key1 = 'a';
```

##### 11-5、Using index condition
如果在查询语句的执行过程中将要使用 `索引下推` 这个特性，在 Extra 列中将会显示 Using indexcondition。比如下边这个查询：
```sql
SELECT * FROM s1 WHERE key1 > 'z' AND key1 LIKE '%a';
```
其中的 key1 > 'z' 可以使用到索引，但是 key1 LIKE '%a' 却无法使用到索引。

##### 11-6、Using where
当某个搜索条件序号在server层进行判断时，在Extra 列中会提示上述额外信息。

```sql
EXPLAIN SELECT * FROM s1 WHERE common_field = 'a';
```

其他例子
```sql
EXPLAIN SELECT * FROM s1 WHERE key1 = 'a' AND common_field = 'a';
```
下面这条sql会使用idx_key1二级搜索，但是由于该搜索不包含common_field列，所以需要存储引擎根据二级搜索记录执行回表操作，并将完整的用户记录返回给server之后，再在server层判断这个条件是否成立。

##### 11-7、Using join buffer (Block Nested Loop)
在连接查询执行过程中，当被驱动表不能有效的利用索引加快访问速度， MySQL 一般会为其分配一块名叫join buffer 的内存块来加快查询速度，也就是我们所讲的 基于块的嵌套循环算法 ，比如下边这个查询语句：

```sql
EXPLAIN SELECT * FROM s1 INNER JOIN s2 ON s1.common_field = s2.common_field;
```
s2表提示： Using where; Using join buffer (Block Nested Loop)

##### 11-8、Using intersect(...) 、 Using union(...) 和 Using sort_union(...)

##### 11-9、Not exists
当我们使用左（外）连接时，如果 WHERE 子句中包含要求被驱动表的某个列等于 NULL 值的搜索条件，而且那个列又是不允许存储 NULL 值的，那么在该表的执行计划的 Extra 列就会提示 Not exists 额外信息，比如这样：

```sql
EXPLAIN SELECT * FROM s1 LEFT JOIN s2 ON s1.key1 = s2.key1 WHERE s2.id IS NULL
```

##### 11-10、Using filesort

##### 11-11、Using temporary

##### 11-11、Start temporary, End temporary

##### 11-11、LooseScan

##### 11-11、FirstMatch(tbl_name)

## Json格式的执行计划
查看某个执行计划花费的成本的方式：在 EXPLAIN 单词和真正的查询语句中间加上 FORMAT=JSON 。

```sql
EXPLAIN FORMAT=JSON SELECT * FROM s1 INNER JOIN s2 ON s1.key1 = s2.key2 WHERE s1.common_field = 'a'\G
```









 


