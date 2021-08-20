### SADD
* SADD key member [member ...]
* O(N)， N 是被添加的元素的数量
* 返回：被添加到集合中的新元素的数量，不包括被相同而被忽略的元素

### SCARD
* 返回集合 key 的基数(集合中元素的数量)


### SDIFF
* SDIFF key [key ...]
* 返回第一个集合且不在后面集合中的元素
* O(N)， N 是所有给定集合的成员数量之和
* 返回：差集

### SDIFFSTORE
* SDIFFSTORE destination key [key ...]
* O(N)， N 是所有给定集合的成员数量之和
* 这个命令的作用和 SDIFF 类似，但它将结果保存到 destination(可以和key一样) 集合，并返回结果集

### SINTER
* SINTER key [key ...]
* 返回所有给定集合的交集
* O(N * M)， N 为给定集合当中基数最小的集合， M 为给定集合的个数
* 返回：交集成员的列表。

### SINTERSTORE(同SDIFFSTORE)
* SINTERSTORE destination key [key ...]

### SISMEMBER
* SISMEMBER key member
* 判断 member 元素是否集合 key 的成员。
* 返回：是返回1，否则0

### SMEMBERS
* 返回集合 key 中的所有成员
* O(N)， N 为集合的基数

### SMOVE
* SMOVE source destination member
* O(1)
* 将 member 元素从 source 集合移动到 destination 集合。
* SMOVE 是原子性操作
* 返回：1 或者 0

### SPOP
* 移除并返回集合中的一个随机元素。

### SRANDMEMBER
* SRANDMEMBER key [count]
* 返回集合中的一个随机元素
* 如果命令无count，那么返回集合中的一个随机元素

##### count用法
* 如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合。
* 如果 count 为负数，那么命令返回一个数组，数组中的元素**可能会重复出现多次**，而数组的长度为 count 的绝对值。

### SREM
* SREM key member [member ...]
* O(N)， N 为给定 member 元素的数量
* 返回：成移除元素个数

### SUNION
* SUNION key [key ...]
* 返回一个集合的全部成员，该集合是所有给定集合的并集
* O(N)， N 是所有给定集合的成员数量之和

### SUNIONSTORE
* SUNIONSTORE destination key [key ...]
* O(N)， N 是所有给定集合的成员数量之和。

### SSCAN(见汇总)

