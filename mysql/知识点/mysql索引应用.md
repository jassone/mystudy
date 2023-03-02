## mysql索引使用
## 一、索引的代码
### 1、空间上的代价
这个是显而易见的，每建立一个索引都要为它建立一棵 B+ 树，每一棵 B+ 树的每一个节点都是一个数据页，一个页默认会占用 16KB 的存储空间，一棵很大的 B+ 树由许多数据页组成，那可是很大的一片存储空间呢。

### 2、时间上的代价
每次对表中的数据进行增、删、改操作时，都需要去修改各个 B+ 树索引。而且我们讲过， B+ 树每层节点都是按照索引列的值从小到大的顺序排序而组成了双向链表。不论是叶子节点中的记录，还是内节点中的记录（也就是不论是用户记录还是目录项记录）都是按照索引列的值从小到大的顺序而形成了一个单向链表。而增、删、改操作可能会对节点和记录的排序造成破坏，所以存储引擎需要额外的时间进行一些记录移位，页面分裂、页面回收啥的操作来维护好节点和记录的排序。

另外在执行查询语句前，首先要生成一个执行计划，如果建太多索引，导致成本分析过程耗时太多，从而影响查询语句的执行性能。

## 二、扫描区间和边界条件
在使用某个索引执行查询时，关键的问题就是通过搜索条件找出合适的扫描区间，然后再到对应的B+树中扫描**索引列值**在这些扫描区间的记录。对于每个扫描区间，仅需要通过B+树定位到该扫描区间中的第一条记录，然后就可以沿着记录所在的单向链表向后扫描。

**所以扫描区间的意义在于能走到索引。**

准备一张表，后面会用到
```sql
CREATE TABLE single_table(
 id INT NOT NULL auto_increment,
 key1 VARCHAR(100) ,
 key2 int,
 key2 VARCHAR(100) ,
 key_part1 VARCHAR(100) ,
 key_part2 VARCHAR(100) ,
 key_part3 VARCHAR(100) ,
 common_field VARCHAR(100) ,
 PRIMARY KEY (id),
 KEY idx_key1(key1),
 unique uk_key2(key2),
 KEY idx_key3(key3),
 KEY idx_key_part(key1, key2, key3)
);
```

### 1、任何搜索条件都可以生成合适的扫描区间的情况
```sql
select * from single_table where key2 > 100 and key2 > 200;
```
取交集，扫描区间（200，+∞）

```sql
select * from single_table where key2 > 100 and common_field='a';
```
common_field='a'没有起到任何作用，扫描区间（100，+∞）

### 2、有些搜索条件不能生成合适的扫描区间的情况
```sql
select * from single_table where key2 > 100 or common_field='a';

等同于

select * from single_table where key2 > 100 or true;
```
扫描区间（-∞，+∞）,因为要执行回表查询，比全表扫描代价大，所以不考虑使用索引uk_key2来执行查询。

### 3、使用联合索引执行查询时对应的扫描区间的情况
```sql
select * from single_table where key_part1 = 'a';
```
扫描区间['a','a']

```sql
select * from single_table where key_part1 = 'a' and key_part2 = 'b';
```
扫描区间[（'a','b'），（'a','b'）]

```sql
select * from single_table where key_part1 < 'a' ;
```
扫描区间 (-∞,'a')

```sql
select * from single_table where key_part1 = 'a' and key_part3 = 'c';
```
扫描区间['a','a'],和key_part3无关。但这时候会走到索引下推。

### 4、有些查询需要查询优化器决定使用哪个扫描区间
```sql
select * from single_table where key1 = 'a' and key2 > 'b';
```
扫描区间可能是['a','a']或(b，+∞),由查询优化器决定使用哪个索引。

## 三、索引用于排序
我们在写查询语句的时候经常需要对查询出来的记录通过 ORDER BY 子句按照某种规则进行排序。一般情况下，我们只能把记录都加载到内存中，再用一些排序算法在内存中对这些记录进行排序，有的时候可能查询的结果集太大以至于不能在内存中进行排序的话，还可能暂时借助磁盘的空间来存放中间结果，排序操作完成后再把排好序的结果集返回到客户端。在 MySQL 中，把这种在内存中或者磁盘上进行排序的方式统称为**文件排序**（英文名： filesort ）。

但是如果 ORDER BY 子句里使用到了我们的索引列，就有可能省去在内存或文件中排序的步骤。

比如
```sql
select * from single_table order by key_part1,key_part2, key_part3 limit 10;
```

### 1、使用联合索引进行排序注意事项
对于 联合索引 有个问题需要注意， ORDER BY 的子句后边的列的顺序也必须**按照索引列的顺序**给出。

### 2、不可以使用索引进行排序的几种情况
* ASC、DESC混用
* 排序列包含非同一个索引的列
* 排序列是某个联合索引的索引列，但是这些排序列在联合索引中并不连续。
* 用来形成扫描区间的索引列与排序列不同

    ```sql
    select * from single_table where key1 = 'a'  order by key2 ;
    ```
    
* 排序列不是以单独列名的形式出现在order by子句中 

    ```sql
    select * from single_table  order by upper(key2 ) limit 10;
    ```

## 三、索引用于分组
```sql
select key_part1,key_part2,key_part3,count(*) from single_table group by key_part1,key_part2,key_part13;
```
如果没有idx_key_part索引，就得建立一个用于统计的临时表，在扫描聚簇索引的记录时，将统计的中间结果填入这个临时表，当扫描完记录后，再把临时表中的结果作为结果集发送给客户端。

而如果有了索引的话，恰巧这个分组顺序又和我们的 B+ 树中的索引列的顺序是一致的，而我们的 B+ 树索引又是按照索引列排好序的，这不正好么，所以可以直接使用B+ 树索引进行分组。

注意：**分组列的顺序也需要和索引列的顺序一致，也可以只使用索引列中左边的列进行分组。**

## 四、更好的创建和使用索引
* 只为用于搜索、排序或分组的列创建索引
* 考虑列的基数
    列的基数 指的是**某一列中不重复数据的个数**。在记录行数一定的情况下，列的基数越大，该列中的值越分散，列的基数越小，该列中的值越集中。

    **这个 列的基数 指标非常重要**，直接影响我们是否能有效的利用索引。假设某个列的基数为 1 ，也就是所有记录在该列中的值都一样，那为该列建立索引是没有用的，因为所有值都一样就无法排序，无法进行快速查找了。而且如果某个建立了**二级索引的列的重复值特别多**，那么使用这个二级索引查出的记录还可能要做回表操作，这样性能损耗就更大了。所以结论就是：最好为那些列的基数大的列建立索引，为基数太小列的建立索引效果可能不好

* 索引列的类型尽量小
* 为列前缀建立索引
  
    ```sql
    KEY index (key1(10))
    
    select * from single_table  where key1 = 'abcdefghijklmn' limit 10;
    ```
    则在查询的时候，先定位到前缀abcdefghij的二级索引记录，在扫描这些二级索引记录时再判断他们是否满足key1 = 'abcdefghijklmn'。

* 索引覆盖
* 让索引列以列名的形式在搜索条件中单独出现
    WHERE my_col * 2 < 4    错误
    WHERE my_col < 4/2      正确
    
* 主键按顺序插入（减少页分裂）



