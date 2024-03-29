## mysql索引
## 一、Innodb存储引擎的索引
### 1、主键索引-聚簇索引(聚集索引)

![12456765.png](https://pic.imgdb.cn/item/619f32e92ab3f51d910d92e0.png)

聚簇索引并不是一种单独的索引类型，而是一种**数据存储方式**。具体细节依赖于其实现方式。

Innodb存储引擎，B+Tree索引可以分为聚簇索引（也称聚集索引，clustered index）和辅助索引（有时也称非聚簇索引或二级索引，secondary index，non-clustered index）。**这两种索引内部都是B+Tree**，聚簇索引的叶子节点存放着一整行的数据。也将聚簇索引的叶子节点称为数据页。这个特性决定了**索引组织表中数据也是索引的一部分**。

我们日常工作中，根据实际情况自行添加的索引都是**辅助索引**，辅助索引就是一个**为了寻找主键索引的二级索引**，找到主键索引再通过主键索引找数据。

Innobd中的主键索引是一种聚簇索引，非聚簇索引都是辅助索引，像复合索引、前缀索引、唯一索引。

innodb索引默认是主键id(但是这个主键更改代价较高，故建表时要考虑自增ID不能频繁update这点)。如果没有主键，则会选择一个唯一的非空索引代替。如果还是没有这样的搜索，会在内部生成一个名为GEN_CLUST_INDEX 的隐式的聚簇索引。

详细结构图如下
![20220122172529.jpg](https://pic.imgdb.cn/item/61ebce082ab3f51d91f66201.jpg)

每个页对应一个目录项，目录项包括下面两个部分
* 页的用户记录，即最小的主键值(key)
* 页号(page_no)

一般3层树，大概可存放1亿条数据，4层树，大概可存放千亿记录。所以一般不会超过4层。

##### 优点：
* 相比myisam存储引擎数据访问更快，因为**聚簇索引将索引和数据保存在同一个B+树中**，支持覆盖索引。因此从聚簇索引中获取数据比非聚簇索引更快。
* 聚簇索引对于主键的排序查找和范围查找速度非常快。

##### 缺点：
* **插入速度严重依赖于插入顺序**。按照主键的顺序插入是最快的方式，否则将会出现页分裂，严重影响性能。因此，对于InnoDB表，**一般都会定义一个自增的ID列为主键**。
* **更新主键的代价很高，因为将会导致被更新的行移动**。因此，对于InnoDB表，我们一般定义主键为不可更新。
* **二级索引访问需要两次索引查找**。第一次找到主键值，第二次根据主键值找到行数据。 所以会比myisam慢一点。

### 2、辅助索引-非聚簇索引

在聚簇索引之上创建的索引称之为辅助索引，辅助索引访问数据总是需要二次查找。辅助索引叶子节点存储的不再是行的物理位置，而是**主键值**。通过辅助索引首先找到的是主键值，再通过主键值找到数据行的数据页，再通过数据页中的Page Directory找到槽，再从槽中找数据行。

Innodb辅助索引的叶子节点并不包含行记录的全部数据，叶子节点除了包含键值外，还包含了相应行数据的聚簇索引键。

辅助索引的存在不影响数据在聚簇索引中的组织，所以一张表可以有多个辅助索引。在innodb中有时也称辅助索引为二级索引。

**采用二级索引来执行查询时，其实每获取到一条二级索引记录，就会立刻对其执行回表操作。(实际上有个MRg功能，但是比较苛刻，所以很少用到)**

### 3、注意点
* 为什么不建议使用过长的字段作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大。
* 不要用**非单调**的字段作为主键，因为InnoDB数据文件本身是一颗B+Tree，非单调的主键会造成在插入新记录时数据文件为了维持B+Tree的特性而频繁的分裂调整，十分低效，而使用自增字段作为主键则是一个很好的选择。
* InnoDB不会压缩索引。
* 走辅助索引时，尽量走覆盖索引。
* 尽量不要用select * ,应该写需要的那些字段。

## 二、MyISAM存储引擎的索引
### 主键索引-非聚簇索引
* MyISAM引擎使用B+Tree作为索引结构(包含主键索引和辅助索引)，**叶节点的data域存放的都是数据记录的地址**。
* MyISAM索引文件和数据文件是分离的，表数据存储顺序和索引顺序无关。索引文件仅保存数据记录的地址。

主键索引
![23243534.png](https://pic.imgdb.cn/item/619f8f5b2ab3f51d913b793a.png)

辅助索引
![1464190-20191106151427711-625351515.png](https://pic.imgdb.cn/item/61a059dc2ab3f51d917cc6af.png)

非聚簇索引的两棵B+树看上去没什么不同，节点的结构完全一致只是存储的内容不同而已，主键索引B+树的节点存储了主键，辅助键索引B+树存储了辅助键。表数据存储在独立的地方，这两颗B+树的叶子节点都使用一个地址指向真正的表数据，对于表数据来说，这两个键没有任何差别。**由于索引树是独立的，通过辅助键检索无需访问主键的索引树**。

## 三、Innodb索引和MyISAM索引异同
![1464190-20191106151427711-625351515.png](https://pic.imgdb.cn/item/61a05c172ab3f51d917d7afb.png)

## 四、其他说明
### 1、二级索引排序规则
二级索引的记录是由 索引列 + 主键 构成的，二级索引列的值相同的记录可能会有好多条，这些索引列的值相同的记录又会按照 **主键的值进行排序**。
### 2、B+树比B-树更适合数据库索引？

* **B+树的磁盘读写代价更低**：B+树的内部节点并没有指向关键字具体信息的指针，因此其内部节点相对B树更小，如果把所有同一内部节点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多，一次性读入内存的需要查找的关键字也就越多，相对IO读写次数就降低了。

* **B+树的查询效率更加稳定**：由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

* **B+树的数据都存储在叶子结点中，分支结点均为索引，方便扫库，只需要扫一遍叶子结点即可**。但是B树因为其分支结点同样存储着数据，我们要找到具体的数据，需要进行一次**中序遍历**按序来扫，**所以B+树更加适合在区间查询的情况**。

## 五、相关概念

### 1、最左匹配
![222212.png](https://pic.imgdb.cn/item/61a076012ab3f51d918762c8.png)

图中a相等的情况下，b是有序的，所以为什么联合搜索，要遵循最左前缀法则。

一般是在联合索引情况下会发生索引失效，看下面几个demo:
1. where b=2,走不到搜索
2. where a>1 and b=1，这时候b的查找走不到搜索，因为即使查找到a的范围，但是b因为是无序的，**所以也走不了二分查找**。
3. where a=1 and b=1 都能走到搜索
4. where a like "%1" 不走搜索
5. where a like "1%" 某些情况下走搜索

在 InnoDB 中联合索引只有先确定了前一个（左侧的值）后，才能确定下一个值。如果有范围查询的话，那么联合索引中使用范围查询的字段`后`的索引在该条 SQL 中都不会起作用。

就是当遇到范围查询(>、<、between、like)就会停止，后面的查询将走不到索引。也就是：
```sql
select * from t where a=1 and b>1 and c =1; 
```
这样a,b可以用到（a,b,c），c索引用不到 

> 字符串也是按照顺序排列的

![FA82926D-41B9-41CC-9DB1-B6C7C80547FA.png](https://pic.imgdb.cn/item/61a0779b2ab3f51d9187f951.png)

##### 多列索引注意点
* 多列索引的顺序如何选择
一般情况下，把选择性高的字段放在前面。

* 避免使用范围查询
很多情况下，范围查询都可能导致无法使用索引。
* 尽量避免查询不需要的数据(尽量不要用select * )，尽量走索引覆盖。
* 查询的数据类型要正确
```sql
explain select * from user where create_date >= now();
explain select * from user where create_date >= '2020-05-01 00:00:00';
```
第一条语句就可以使用create_date的索引，第二个就不可以。

### 2、回表
InnoDB的存储引擎下，二级索引无法直接查询所有列的数据，所以通过二级索引查询到聚簇索引后，再查询到想要的数据，这种通过二级索引查询出来的过程，就叫做回表。

索引中的数据页都必须存放在磁盘中，等到需要的时候再加载到内存中使用。当我们每读取一条二级索引记录，就需要根据该二级索引记录的id到聚簇索引中执行回表操作，如果对应的聚簇索引记录所在的页面不在内存中，就需要将也能从磁盘加载到内存中，由于并不连续的 id值到聚簇索引中访问完整的用户记录可能分布在不同的数据页中，这样读取完整的用户记录可能要访问更多的数据页，这种读取方式我们也可以称为 随机I/O 。

回表是循环操作，不是一次性拿到所有数据再一起回表。需要回表的记录越多，使用二级索引的性能就越低，甚至让某些查询宁愿使用全表扫描也不使用 二级索引 。

### 3、索引覆盖
覆盖索引（covering index）指一个查询语句的执行只用从索引中就能够取得，不必从数据表中读取。也可以称之为实现了索引覆盖。

**最好在查询列表中只包含索引列**，这样MySQL只需要通过索引就可以返回查询所需要的数据，这样避免了查到索引后再返回表操作，减少I/O提高效率。

##### 前面有%的查询如何走索引

比如给name设置索引，这样查询，会走到索引，因为走了索引覆盖

```sql
select name from t where name like "%abc%"
```

联合索引也是同样的道理。

### 4、索引下推

详见：`索引下推`单篇文章。

