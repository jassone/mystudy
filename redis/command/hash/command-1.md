### HDEL
* 原子操作，低版本用MULTI/EXEC ???
* O(N)， N 为要删除的域的数量
* 返回：被成功移除的域的数量，不包括被忽略的域


### HEXISTS
* 查看哈希表 key 中，给定域 field 是否存在

### HGET

### HGETALL
* O(N)， N 为哈希表的大小。
* 返回：若key不存在，返回空列表

### HINCRBY
* HINCRBY key field increment
* 增量也可以为负数，相当于对给定域进行减法操作
* 如果 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令
* 如果域 field 不存在，那么在执行命令前，域的值被初始化为 0 
* 对一个储存字符串值的域 field 执行 HINCRBY 命令将造成一个错误
* 本操作的值被限制在 64 位(bit)有符号数字表示之内
* 返回：执行 HINCRBY 命令之后，哈希表 key 中域 field 的值

### HINCRBYFLOAT
* HINCRBYFLOAT key field increment
* 当无key或者field时，同HINCRBY处理

    ##### 当以下任意一个条件发生时，返回一个错误：
* 域 field 的值不是字符串类型(因为 redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
* 域 field 当前的值或给定的增量 increment 不能解释(parse)为双精度浮点数(double precision floating point number)

### HKEYS
* O(N)， N 为哈希表的大小
* 返回：一个包含哈希表中所有域的表。当 key 不存在时，返回一个空表


### HLEN
* 返回：哈希表中域的数量。当 key 不存在时，返回0

### HMGET
* O(N)， N 为给定域的数量
* 返回：一个包含多个给定域的关联值的表，表值的排列顺序和给定域参数的请求顺序一样

### HMSET
* O(N)， N 为 field-value 对的数量
* 如果 key 不存在，一个空哈希表被创建并执行 HMSET 操作
* 返回：如果命令执行成功，返回 OK 

### HSET
* 如果 key 不存在，一个新的哈希表被创建并进行 HSET 操作
* 返回：<font color="red">如果 field 是哈希表中的一个新建域，并且值设置成功，返回 1 。如果哈希表中域 field 已经存在且旧值已被新值覆盖，返回 0 </font>

### HSETNX
* 返回：设置成功，返回 1 。
如果给定域已经存在且没有操作被执行，返回 0 


### HVALS
* O(N)， N 为哈希表的大小
* 返回：一个包含哈希表中所有值的表。当 key 不存在时，返回一个空表

### HSCAN（见汇总）







