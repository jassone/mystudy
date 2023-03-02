### DEL
* DEL key [key ...]
* O(N)， N 为被删除的 key 的数量。删除单个字符串类型的 key ，时间复杂度为O(1)。删除单个列表、集合、有序集合或哈希表类型的 key ，时间复杂度为O(M)， M 为以上数据结构内的元素数量。
* 返回：被删除 key 的数量。

### UNLINK
与DEL 命令类似。但是 UNLINK 命令不同与 DEL 命令在于它是**异步执行的，因此它不会阻塞**。UNLINK 命令是非阻塞删除，就是将删除操作放到另外一个线程（而非 Redis 主线程）去处理。

* 可用版本：>=4.0.0
* 时间复杂度：当删除每个指定 key 的时候，时间复杂度为 O(1)，不管 key 值多大；然后在另一个线程回收内存操作时候，时间复杂度为 O(N)，其中 N 为分配给删除对象值的内存分配数。
* 返回可删除 key 的个数。

### DUMP
* DUMP key
* 序列化给定 key ，并返回被序列化的值，使用 RESTORE 命令可以将这个值反序列化为 Redis 键。
* 查找给定键的复杂度为 O(1) ，对键进行序列化的复杂度为 O(N*M) ，其中 N 是构成 key 的 Redis 对象的数量，而 M 则是这些对象的平均大小。如果序列化的对象是比较小的字符串，那么复杂度为 O(1) 

##### 特点
* 它带有 64 位的校验和，用于检测错误， RESTORE 在进行反序列化之前会先检查校验和。
* 值的编码格式和 RDB 文件保持一致。
* RDB 版本会被编码在序列化值当中，如果因为 Redis 的版本不同造成 RDB 格式不兼容，那么 Redis 会拒绝对这个值进行反序列化操作

##### 模型
* 一个典型的应用是key的迁移与还原

###  RESTORE
* RESTORE key ttl serialized-value
* 反序列化给定的序列化值，并将它和给定的 key 关联。
* 参数 ttl 以毫秒为单位为 key 设置生存时间；如果 ttl 为 0 ，那么不设置生存时间。

* RESTORE 在执行反序列化之前会先对序列化值的 RDB 版本和数据校验和进行检查，如果 RDB 版本不相同或者数据不完整的话，那么 RESTORE 会拒绝进行反序列化，并返回一个错误

##### 时间复杂度
* 查找给定键的复杂度为 O(1) ，对键进行反序列化的复杂度为 O(N*M) ，其中 N 是构成 key 的 Redis 对象的数量，而 M 则是这些对象的平均大小
* 有序集合(sorted set)的反序列化复杂度为 O(N*M*log(N)) ，因为有序集合每次插入的复杂度为 O(log(N))
* 如果反序列化的对象是比较小的字符串，那么复杂度为 O(1)


### EXISTS

### EXPIRE
* EXPIRE key seconds
* 设置过期时间-秒
* 过期时间精确度：在 Redis 2.4 版本中，过期时间的延迟在 1 秒钟之内 —— 也即是，就算 key 已经过期，但它还是可能在过期之后一秒钟之内被访问到，而在新的 Redis 2.6 版本中，延迟被降低到 1 毫秒之内
* <font color="red">为负数则删除该key</font>

### PEXPIRE(同上，毫秒级别)
* PEXPIRE key milliseconds

### EXPIREAT
* EXPIREAT key timestamp
* 设置过期时间-时间戳
* 接受的时间参数是 UNIX 时间戳(unix timestamp)
* <font color="red">过去时间则删除该key</font>

### PEXPIREAT(同上，毫秒级别)
* PEXPIREAT key milliseconds-timestamp

### PERSIST
* 移除给定 key 的生存时间，设置为永久有效


### KEYS
* KEYS pattern
* 查找所有符合给定模式 pattern 的 key 
* <font color="red">KEYS 的速度非常快，但在一个大的数据库中使用它仍然可能造成性能问题，如果你需要从一个数据集中查找特定的 key ，你最好还是用 Redis 的集合结构(set)来代替</font>
* O(N)， N 为数据库中 key 的数量

### MIGRATE
* MIGRATE host port key destination-db timeout [COPY] [REPLACE]
* key 数据在两个实例之间传输的复杂度为 O(N) 
* 将 key 原子性地从当前实例传送到目标实例的指定数据库上，一旦传送成功， key 保证会出现在目标实例上，而当前实例上的 key 会被删除。

这个命令是一个原子操作，它在执行的时候会阻塞进行迁移的两个实例，直到以下任意结果发生：迁移成功，迁移失败，等到超时。

命令的内部实现是这样的：它在当前实例对给定 key 执行 DUMP 命令 ，将它序列化，然后传送到目标实例，目标实例再使用 RESTORE 对数据进行反序列化，并将反序列化所得的数据添加到数据库中；当前实例就像目标实例的客户端那样，只要看到 RESTORE 命令返回 OK ，它就会调用 DEL 删除自己数据库上的 key 。

timeout 参数以毫秒为格式，指定当前实例和目标实例进行沟通的最大间隔时间。这说明操作并不一定要在 timeout 毫秒内完成，只是说数据传送的时间不能超过这个 timeout 数。

MIGRATE 命令需要在给定的时间规定内完成 IO 操作。如果在传送数据时发生 IO 错误，或者达到了超时时间，那么命令会停止执行，并返回一个特殊的错误： IOERR 。

当 IOERR 出现时，有以下两种可能：

* key 可能存在于两个实例
* key 可能只存在于当前实例

唯一不可能发生的情况就是丢失 key ，因此，如果一个客户端执行 MIGRATE 命令，并且不幸遇上 IOERR 错误，那么这个客户端唯一要做的就是检查自己数据库上的 key 是否已经被正确地删除。

如果有其他错误发生，那么 MIGRATE 保证 key 只会出现在当前实例中。（当然，目标实例的给定数据库上可能有和 key 同名的键，不过这和 MIGRATE 命令没有关系）。

可选项：
* COPY ：不移除源实例上的 key 。
* REPLACE ：替换目标实例上已存在的 key 。

### MOVE
* MOVE key db
* 将当前数据库的 key 移动到给定的数据库 db 当中
* 如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key ，或者 key 不存在于当前数据库，那么 MOVE 没有任何效果。因此，也可以利用这一特性，<font color="red">将 MOVE 当作锁(locking)原语(primitive) ？？？？？</font>

### OBJECT
* OBJECT subcommand [arguments [arguments]]
* OBJECT 命令允许从内部察看给定 key 的 Redis 对象

##### 用途
* 它通常用在除错(debugging)或者了解为了节省空间而对 key 使用特殊编码的情况。
* 当将Redis用作缓存程序时，你也可以通过 OBJECT 命令中的信息，决定 key 的驱逐策略(eviction policies)。

##### OBJECT 命令有多个子命令：

* OBJECT REFCOUNT <key> 返回给定 key 引用所储存的值的次数。此命令主要用于除错。
* OBJECT ENCODING <key> 返回给定 key 锁储存的值所使用的内部表示(representation)。
* OBJECT IDLETIME <key> <font color="red">返回给定 key 自储存以来的空转时间(idle， 没有被读取也没有被写入)，以秒为单位</font>

##### 对象可以以多种方式编码：
* 字符串可以被编码为 raw (一般字符串)或 int (用字符串表示64位数字是为了节约空间)。
* 列表可以被编码为 ziplist 或 linkedlist 。 ziplist 是为节约大小较小的列表空间而作的特殊表示。
* 集合可以被编码为 intset 或者 hashtable 。 intset 是只储存数字的小集合的特殊表示。
* 哈希表可以编码为 zipmap 或者 hashtable 。 zipmap 是小哈希表的特殊表示。
* * 有序集合可以被编码为 ziplist 或者 skiplist 格式。 ziplist 用于表示小的有序集合，而 skiplist 则用于表示任何大小的有序集合。
* 假如你做了什么让 Redis 没办法再使用节省空间的编码时(比如将一个只有 1 个元素的集合扩展为一个有 100 万个元素的集合)，特殊编码类型(specially encoded types)会自动转换成通用类型(general type)。


### TTL
* 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)

#####  返回
* 当 key 不存在时，返回 -2 。
* 当 key 存在但没有设置剩余生存时间时，返回 -1 。
* 否则，以秒为单位，返回 key 的剩余生存时间。

### PTTL(同上，毫秒级别)

### RANDOMKEY
* 从当前数据库中随机返回(不删除)一个 key 

### RENAME
* RENAME key newkey
* 当 key 和 newkey 相同，或者 key 不存在时，返回一个错误
* 当 newkey 已经存在时， RENAME 命令将覆盖旧值
* <font color="red">newkey的ttl是老的key的</font>

##### 模式
* 比如hash类型的值在批量设置时，可以用rename达到原子操作

### RENAMENX（同上）
* RENAMENX key newkey
* 当且仅当 newkey 不存在时，将 key 改名为 newkey 

### SORT
* SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]
* 返回或保存给定列表、集合、有序集合 key 中经过排序的元素。
* <font color="red">排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较</font>
* SORT website ALPHA ,按字符排序
* 其他排序见文档

##### 模式
* 可以通过将 SORT 命令的执行结果保存，并用 EXPIRE 为结果设置生存时间，以此来产生一个 SORT 操作的结果缓存。这样就可以避免对 SORT 操作的频繁调用：只有当结果集过期时，才需要再调用一次 SORT 操作。
* 另外，为了正确实现这一用法，你可能需要加锁以避免多个客户端同时进行缓存重建(也就是多个客户端，同一时间进行 SORT 操作，并保存为结果集)，具体参见 SETNX 命令。

### TYPE
* 返回 key 所储存的值的类型

### SCAN









