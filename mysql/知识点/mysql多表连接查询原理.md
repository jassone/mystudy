## mysql多表连接查询原理
连接 的本质就是把各个连接表中的记录都取出来进行依次匹配，并把匹配后的组合加入结果集并返回给用户。

## 一、连接过程简介

```sql
SELECT * FROM t1, t2 WHERE t1.m1 > 1 AND t1.m1 = t2.m2 AND t2.n2 < 'd';
```

### 1、过程
* 首先确定第一个需要查询的表，这个表称之为 `驱动表`。
* 针对上一步骤中从驱动表产生的结果集中的每一条记录，分别需要到 t2 表中查找匹配的记录。t2 表也可以被称之为 `被驱动表`。

### 2、说明
* 在两表连接查询中，驱动表只需要访问一次，被驱动表可能被访问多次。
* 每次获取到一条驱动表记录，就立刻到被驱动表中寻找匹配的记录。

## 二、内连接和外连接

```sql
SELECT * FROM student AS s1, score AS s2 WHERE s1.number = s2.number;
```
上面查询的结果是如果某个表中匹配不到，则另外一个表中的记录也不会返回。

### 1、 内连接和外连接定义
如果我们想驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集。为了解决这个问题，就有了 `内连接` 和 `外连接` 的概念：
* 对于 内连接 的两个表，驱动表中的记录在被驱动表中找不到匹配的记录，该记录不会加入到最后的结果集，我们上边提到的连接都是所谓的 内连接 。
* 对于 外连接 的两个表，驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集。

    在 MySQL 中，根据选取驱动表的不同，外连接仍然可以细分为2种：
    * 左外连接:选取左侧的表为驱动表。
    * 右外连接:选取右侧的表为驱动表。

### 2、WHERE 子句中的过滤条件
WHERE 子句中的过滤条件就是我们平时见的那种，不论是内连接还是外连接，凡是不符合 WHERE 子句中的过滤条件的记录都不会被加入最后的结果集。
### 2、ON 子句中的过滤条件
内连接中的WHERE子句和ON子句是等价的。  

对于外连接的驱动表的记录来说，如果无法在被驱动表中找到匹配 ON 子句中的过滤条件的记录，那么该记录仍然会被加入到结果集中，对应的被驱动表记录的各个字段使用 NULL 值填充。

## 三、嵌套循环连接（Nested-Loop Join）
这种驱动表只访问一次，但被驱动表却可能被多次访问，访问次数取决于对驱动表执行单表查询后的结果集中的记录条数的连接执行方式称之为 嵌套循环连接 （ Nested-Loop Join ），这是最简单，也是最笨拙的一种连接查询算法。

### 1、使用索引加快连接速度
### 2、基于块的嵌套循环连接（Block Nested-Loop Join）
join buffer 就是执行连接查询前申请的一块固定大小的内存，先把若干条驱动表结果集中的记录装在这个 join buffer 中，然后开始扫描被驱动表，每一条被驱动表的记录一次性和 join buffer 中的多条驱动表记录做匹配，因为匹配的过程都是在内存中完成的，所以这样可以显著减少被驱动表的 I/O 代价。

另外需要注意的是，驱动表的记录并不是所有列都会被放到 join buffer 中，只有**查询列表中的列**和**过滤条件中的列**才会被放到 join buffer 中，所以再次提醒我们，最好不要把 * 作为查询列表，只需要把我们关心的列放到查询列表就好了，这样还可以在 join buffer 中放置更多的记录呢哈。