### set
##### 特性

* 当 SET 命令对一个带有生存时间（TTL）的键进行设置之后(<font color="red">不管value是否发生变化</font>)， 该键原有的 TTL 将被清除
* 从 Redis 2.6.12 版本开始， SET 在设置操作成功完成时，才返回 OK 。如果设置了 NX 或者 XX ，但因为条件没达到而造成设置操作未执行，那么命令返回空批量回复（NULL Bulk Reply）。
* NX 或 XX 可以和 EX 或者 PX 组合使用


##### 参数
* 1. EX second ：设置键的过期时间为 second 秒。 SET key value EX second 效果等同于 SETEX key second value 。
* 2. PX millisecond ：设置键的过期时间为 millisecond 毫秒。 SET key value PX millisecond 效果等同于 PSETEX key millisecond value 。
* 3. NX ：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value 。
* 4. XX ：只在键已经存在时，才对键进行设置操作。

##### 模式
> **命令 SET resource-name anystring NX EX max-lock-time 是一种在 Redis 中实现锁的简单方法，表示当前是否有人在用(包括自己)**。

客户端执行以上的命令：
* 如果服务器返回 OK ，那么这个客户端获得锁。
* 如果服务器返回 NIL ，那么客户端获取锁失败，可以在稍后再重试。
* 设置的过期时间到达之后，锁将自动释放。

可以通过以下修改，让这个锁实现更健壮：
* 不使用固定的字符串作为键的值，而是设置一个不可猜测（non-guessable）的长随机字符串，作为口令串（token）。
* 不使用 DEL 命令来释放锁，而是发送一个 Lua 脚本，这个脚本只在客户端传入的值和键的口令串相匹配时，才对键进行删除。


### SETNX 

### SETEX 
* SETEX 和set+expire的不同之处在于 SETEX 是一个原子（atomic）操作

### PSETEX 
* psetex key milliseconds value： 毫秒级别

### GET
* 如果键 key 的值并非字符串类型， 那么返回一个错误， 因为 GET 命令只能用于字符串值。（**比如 LPUSH key redis mongodb mysql 后再get**）

### GETSET 
* getset key value ：设置并返回设置之前的值(若key不存在返回nil)


### STRLEN
* STRLEN key
* <font color="red">空字符串或者不存在返回0</font>


### APPEND 
* APPEND key value
* 将value追加到key值的后面(不存在相当于set)
* 返回设置后的字符串长度

##### 模式
* APPEND 可以为一系列**定长(fixed-size)数据**(sample)提供一种紧凑的表示方式，通常称之为时间序列或者<font color="red">节拍序列</font>。
* STRLEN 给出时间序列中数据的数量
* GETRANGE 可以用于随机访问。只要有相关的时间信息的话，我们就可以在 Redis 2.6 中使用 Lua 脚本和 GETRANGE 命令实现二分查找(<font color="red">??????</font>)。
* SETRANGE 可以用于覆盖或修改已存在的的时间序列。
* 可以考虑使用 UNIX 时间戳作为时间序列的键名，这样一来，可以避免单个 key 因为保存过大的时间序列而占用大量内存，另一方面，<font color="red">也可以节省下大量命名空间??????</font>。

### SETRANGE 
* SETRANGE key offset value
* 小字符串O(1)，否则为O(N)， N 为 value 参数的长度
* 返回被修改之后， 字符串值的长度。
* 如果键 key 原来储存的字符串长度比偏移量小，那么原字符和偏移量之间的空白将用零字节(zerobytes, "\x00" )进行填充（<font color="red">每一个'\u0000'都代表了一个空格 ？？？？</font>）。
* 当生成一个很长的字符串时， Redis 需要分配内存空间， 该操作有时候可能会造成服务器阻塞(block)。 在2010年出产的Macbook Pro上， 设置偏移量为 536870911(512MB 内存分配)将耗费约 300 毫秒， 设置偏移量为 134217728(128MB 内存分配)将耗费约 80 毫秒， 设置偏移量 33554432(32MB 内存分配)将耗费约 30 毫秒， 设置偏移量为 8388608(8MB 内存分配)将耗费约 8 毫秒。
  
### INCR
* INCR key
* 键key不存在，则值会先被初始化为0，再执行INCR
* INCR命令是一个针对字符串的操作。因为 Redis 并没有专用的整数类型，所以键key储存的值在执行 INCR 命令时会被解释为十进制64位有符号整数

##### 模式
* 计数器
* 限数器（用事务打包执行INCR和EXPIRE，避免引入竞争条件）
  
    
### INCRBY
* INCRBY key increment

### INCRBYFLOAT
* 无论是 key 的值，还是增量 increment ，都可以使用像 2.0e7 、 3e5 、 90e-2 那样的指数符号(exponential notation)来表示，但是，执行 INCRBYFLOAT 命令之后的值总是以同样的形式储存，也即是，它们总是由一个数字，一个（可选的）小数点和一个任意位的小数部分组成（比如 3.14 、 69.768 ，诸如此类)，小数部分尾随的 0 会被移除，如果有需要的话，还会将浮点数改为整数（比如 3.0 会被保存成 3 ）
* 最多只能表示小数点的后十七位

### DECR
* DECR key

### DECRBY
*  DECRBY key decrement

### MSET
*  MSET key value [key value …]
*  O(N)，其中 N 为被设置的键数量
*  总是返回 OK 
*  <font color="red">MSET 是一个原子性(atomic)操作， 所有给定键都会在同一时间内被设置， 不会出现某些键被设置了但是另一些键没有被设置的情况</font>

### MSETNX 
* MSETNX key value [key value …]
* O(N)， 其中 N 为被设置的键数量。
* 当且仅当<font color="red">所有</font>给定键都不存在时， 为所有给定键设置值。
* <font color="red">原子性(atomic)操作， 所有给定键要么就全部都被设置， 要么就全部都不设置， 不可能出现第三种状态。</font>
* 成功返回1,如果因为某个给定键已经存在而导致设置未能成功执行,返回0

### MGET
* MGET key [key …]
*  O(N) ，其中 N 为给定键的数量

### GETRANGE
* GETRANGE key start end
* O(N)， N 为要返回的字符串的长度，但因为从已有字符串中取出子字符串的操作非常廉价，所以对于长度不大的字符串，也可看作O(1)。
* 负数偏移量表示从字符串最后开始计数， 比如-1 表示最后一个字符
* 值域超过部分自动被符略

### SETBIT
* SETBIT key offset value
* O(1)
* 返回：指定偏移量原来储存的位
* 对 =key所储存的字符串值，设置或清除指定偏移量上的位(bit)
* 当key不存在时，自动生成一个新的字符串值
* 当offset超过时，空白位置以0填充
* offset 参数必须大于或等于0，小于 2^32 (bit 映射被限制在512MB之内)

### GETBIT
* GETBIT key offset
* 当 offset 比字符串值的长度大，或者 key 不存在时，返回 0 

### BITCOUNT
* BITCOUNT key [start] [end]
* O(N)
* 返回：被设置为1的位的数量
* 注意：<font color="red">start、end 是指bit组的字节的下标</font>
* 不存在的key被当成是空字符串来处理

##### 模式
 使用<font color="red">bitmap</font>实现用户上线次数统计
* 今天是网站上线的第100天，而用户peter在今天阅览过网站，那么执行命令 SETBIT peter 100 1
* 执行 BITCOUNT peter ，得出的结果就是 peter 上线的总天数


##### 性能
即使运行 10 年，占用的空间也只是每个用户 10*365 比特位(bit)，也即是每个用户 456 字节。对于这种大小的数据来说，处理速度非常快。

如果你的 bitmap 数据非常大，那么可以考虑使用以下两种方法：
* 将一个大的 bitmap 分散到不同的 key 中，作为小的 bitmap 来处理。使用 Lua 脚本可以很方便地完成这一工作。
* 使用 BITCOUNT 的 start 和 end 参数，每次只对所需的部分位进行计算，将位的累积工作(accumulating)放到客户端进行，并且对结果进行缓存 (caching)

### BITOP
* O(N)
* 返回：保存到 destkey 的字符串的长度，和输入 key 中最长的字符串长度相等。
* BITOP AND destkey key [key ...] ，对一个或多个 key 求逻辑并，并将结果保存到 destkey 。
* BITOP OR destkey key [key ...] ，对一个或多个 key 求逻辑或，并将结果保存到 destkey 。
* BITOP XOR destkey key [key ...] ，对一个或多个 key 求逻辑异或，并将结果保存到 destkey 。
* BITOP NOT destkey key ，对给定 key 求逻辑非，并将结果保存到 destkey 。
* 除了 NOT 操作之外，其他操作都可以接受一个或多个 key 作为输入。
* 当处理大型矩阵(matrix)或者进行大数据量的统计时，最好将任务指派到附属节点(slave)进行，避免阻塞主节点。？？？？？






​    







