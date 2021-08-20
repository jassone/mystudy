### ZADD
* ZADD key score member [[score member] [score member] ...]
* O(M*log(N))， N 是有序集的基数， M 为成功添加的新成员的数量。
* 如果某个 member 已经是有序集的成员，那么更新这个 member 的 score 值，并通过重新插入这个 member 元素，来保证该 member 在正确的位置上。
* score 值可以是整数值或双精度浮点数
* 返回：被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员
* 在通过ZADD命令添加第一个元素到空 key 时,Redis 会通过检查输入的第一个元素来决定使用何种结构

### ZCARD
* 	获取成员数

### ZCOUNT
* ZCOUNT key min max
* 返回有序集 key 中， score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max )的成员的数量。
* O(log(N)+M)， N 为有序集的基数， M 为值在 min 和 max 之间的元素的<font color="red">数量</font>

### ZINCRBY
* ZINCRBY key increment member
* 为有序集 key 的成员 member 的 score 值加上增量 increment 。可以通过传递一个负数值 increment ，让 score 减去相应的
* O(log(N))
* 返回：member 成员的新 score 值，以字符串形式表示

### ZRANGE
* ZRANGE key start stop [WITHSCORES]
* 返回有序集 key 中，指定区间内的成员。其中成员的位置按 score 值递增(从小到大)来排序。
* 注意：start ，stop是下标(从0开始)
* 如果你需要成员按 score 值递减(从大到小)来排列，请使用 ZREVRANGE 命令。
* O(log(N)+M)， N 为有序集的基数，而 M 为结果集的基数。
* 超出范围的下标并不会引起错误，和其他命令类似

### ZREVRANGE(同ZRANGE)
* ZREVRANGE key start stop [WITHSCORES]
 
### ZRANGEBYSCORE 
* ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
* 其中成员的位置按 score 值递减(从大到小)来排列。
* 具有相同 score 值的成员按字典序(lexicographical order)来排列(该属性是有序集提供的，不需要额外的计算)。
* 可选的 LIMIT 参数指定返回结果的数量及区间(就像SQL中的 SELECT LIMIT offset, count )，<font color="red">注意当 offset 很大时，定位 offset 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间, 这时可考虑用zrange替代</font>

##### 区间及无限

* min 和 max 可以是 -inf 和 +inf ，这样一来，你就可以在不知道有序集的最低和最高 score 值的情况下，使用 ZRANGEBYSCORE 这类命令。

* 默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 <font color="red">(</font> 符号来使用可选的开区间 (小于或大于)。
* demo1:ZRANGEBYSCORE salary (5000 400000   
* demo2:ZRANGEBYSCORE salary -inf 5000 WITHSCORES

### ZREVRANGEBYSCORE(同ZRANGEBYSCORE)
* ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]

### ZRANK
* ZRANK key member
* O(log(N))
* 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大)顺序排列
### ZREVRANK（同ZRANK）
* ZREVRANK key member
*

### ZREM
* ZREM key member [member ...]
* 移除有序集 key 中的一个或多个成员，不存在的成员将被忽略
* O(M*log(N))， N 为有序集的基数， M 为被成功移除的成员的数量
* 返回：被成功移除的成员的数量，不包括被忽略的成员

### ZREMRANGEBYRANK
* ZREMRANGEBYRANK key start stop
* 移除有序集 key 中，指定排名(rank)区间内的所有成员
* O(log(N)+M)， N 为有序集的基数，而 M 为被移除成员的数量
* 返回：被移除成员的数量

### ZREMRANGEBYSCORE
* ZREMRANGEBYSCORE key min max
* 移除有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员
* O(log(N)+M)， N 为有序集的基数，而 M 为被移除成员的数量
* 返回：被移除成员的数量

### ZSCORE
* ZSCORE key member
* 返回有序集 key 中，成员 member 的 score 值

### ZUNIONSTORE
* ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
* 计算给定的一个或多个有序集的**并集**，其中给定 key 的数量必须以 numkeys 参数指定，并将该并集(结果集)储存到 destination 。
* O(N)+O(M log(M))， N 为给定有序集基数的总和， M 为结果集的基数
* 默认情况下，结果集中某个成员的 score 值是所有给定集下该成员 score 值之 和 。

##### WEIGHTS
* 使用 WEIGHTS 选项，你可以为 每个 给定有序集 分别 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 score 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。
* 如果没有指定 WEIGHTS 选项，乘法因子默认设置为 1 。
##### AGGREGATE
* 使用 AGGREGATE 选项，你可以指定并集的结果集的聚合方式。
* 默认使用的参数 SUM ，可以将所有集合中某个成员的 score 值之 和 作为结果集中该成员的 score 值；使用参数 MIN ，可以将所有集合中某个成员的 最小 score 值作为结果集中该成员的 score 值；而参数 MAX 则是将所有集合中某个成员的 最大 score 值作为结果集中该成员的 score 值
 
### ZINTERSTORE(同ZUNIONSTORE)
* ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
* 计算给定的一个或多个有序集的**交集**，其中给定 key 的数量必须以 numkeys 参数指定，并将该交集(结果集)储存到 destination 
* O(N*K)+O(M*log(M))， N 为给定 key 中基数最小的有序集， K 为给定有序集的数量， M 为结果集的基数。

### ZSCAN(见汇总）




