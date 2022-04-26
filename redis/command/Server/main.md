### BGREWRITEAOF
* BGREWRITEAOF
* 执行一个 AOF文件 重写操作。重写会创建一个当前 AOF 文件的体积优化版本。

即使 BGREWRITEAOF 执行失败，也不会有任何数据丢失，因为旧的 AOF 文件在 BGREWRITEAOF 成功之前不会被修改。

重写操作只会在没有其他持久化工作在后台执行时被触发，也就是说：

* 如果 Redis 的子进程正在执行快照的保存工作，那么 AOF 重写的操作会被预定(scheduled)，等到保存工作完成之后再执行 AOF 重写。在这种情况下， BGREWRITEAOF 的返回值仍然是 OK ，但还会加上一条额外的信息，说明 BGREWRITEAOF 要等到保存操作完成之后才能执行。在 Redis 2.6 或以上的版本，可以使用 INFO 命令查看 BGREWRITEAOF 是否被预定。

* 如果已经有别的 AOF 文件重写在执行，那么 BGREWRITEAOF 返回一个错误，并且这个新的 BGREWRITEAOF 请求也不会被预定到下次执行。

从 Redis 2.4 开始， AOF 重写由 Redis 自行触发， BGREWRITEAOF 仅仅用于手动触发重写操作。


### BGSAVE
* O(N)， N 为要保存到数据库中的 key 的数量。
* 在后台异步(Asynchronously)保存当前数据库的数据到磁盘。
* BGSAVE 命令执行之后立即返回 OK ，然后 Redis fork 出一个新子进程，原来的 Redis 进程(父进程)继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。
* 客户端可以通过 LASTSAVE 命令查看相关信息，判断 BGSAVE 命令是否执行成功。

### CLIENT SETNAME
* CLIENT SETNAME connection-name

为当前连接分配一个名字。

这个名字会显示在 CLIENT LIST 命令的结果中， 用于识别当前正在与服务器进行连接的客户端。

举个例子， 在使用 Redis 构建队列（queue）时， 可以根据连接负责的任务（role）， 为信息生产者（producer）和信息消费者（consumer）分别设置不同的名字。

名字使用 Redis 的字符串类型来保存， 最大可以占用 512 MB 。 另外， 为了避免和 CLIENT LIST 命令的输出格式发生冲突， 名字里不允许使用空格。

要移除一个连接的名字， 可以将连接的名字设为空字符串 "" 。

使用 CLIENT GETNAME 命令可以取出连接的名字。

新创建的连接默认是没有名字的。

在 Redis 应用程序发生连接泄漏时，为连接设置名字是一种很好的 debug 手段。

### CLIENT GETNAME
* 返回 CLIENT SETNAME 命令为连接设置的名字。
* 因为新创建的连接默认是没有名字的， 对于没有名字的连接， CLIENT GETNAME 返回空白回复。

### CLIENT LIST
* 以人类可读的格式，返回所有连接到服务器的客户端信息和统计数据。


```
redis> CLIENT LIST
addr=127.0.0.1:43143 fd=6 age=183 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
addr=127.0.0.1:43163 fd=5 age=35 idle=15 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
addr=127.0.0.1:43167 fd=7 age=24 idle=6 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

以下是域的含义：

* addr ： 客户端的地址和端口
* fd ： 套接字所使用的文件描述符
* age ： 以秒计算的已连接时长
* idle ： 以秒计算的空闲时长
* flags ： 客户端 flag （见下文）
* db ： 该客户端正在使用的数据库 ID
* sub ： 已订阅频道的数量
* psub ： 已订阅模式的数量
* multi ： 在事务中被执行的命令数量
* qbuf ： 查询缓存的长度（ 0 表示没有查询在等待）
* qbuf-free ： 查询缓存的剩余空间（ 0 表示没有剩余空间）
* obl ： 输出缓存的长度
* oll ： 输出列表的长度（当输出缓存没有剩余空间时，回复被入队到这个队列里）
* omem ： 输出缓存的内存占用量
* events ： 文件描述符事件（见下文）
* cmd ： 最近一次执行的命令
客户端 flag 可以由以下部分组成：

* O ： 客户端是 MONITOR 模式下的附属节点（slave）
* S ： 客户端是一般模式下（normal）的附属节点
* M ： 客户端是主节点（master）
* x ： 客户端正在执行事务
* b ： 客户端正在等待阻塞事件
* i ： 客户端正在等待 VM I/O 操作（已废弃）
* d ： 一个受监视（watched）的键已被修改， EXEC 命令将失败
* c : 在将回复完整地写出之后，关闭链接
* u : 客户端未被阻塞（unblocked）
* A : 尽可能快地关闭连接
* N : 未设置任何 flag


文件描述符事件可以是：
* r : 客户端套接字（在事件 loop 中）是可读的（readable）
* w : 客户端套接字（在事件 loop 中）是可写的（writeable）


### CLIENT KILL
* CLIENT KILL ip:port

关闭地址为 ip:port 的客户端。
ip:port 应该和 CLIENT LIST 命令输出的其中一行匹配。

因为 Redis 使用单线程设计，所以当 Redis 正在执行命令的时候，不会有客户端被断开连接。

如果要被断开连接的客户端正在执行命令，那么当这个命令执行之后，在发送下一个命令的时候，它就会收到一个网络错误，告知它自身的连接已被关闭。


### CONFIG SET
* CONFIG SET parameter value

CONFIG SET 命令可以动态地调整 Redis 服务器的配置(configuration)而无须重启(但是重启后就失效，所以还是要同步修改配置文件夹)。

CONFIG SET 可以修改的配置参数可以使用命令 CONFIG GET * 来列出，所有被 CONFIG SET 修改的配置参数都会立即生效。


### CONFIG GET
* CONFIG GET parameter

CONFIG GET 命令用于取得运行中的 Redis 服务器的配置参数(configuration parameters) 。

比如执行 CONFIG GET * 命令，服务器就会返回所有配置参数及参数的值

### CONFIG RESETSTAT
重置 INFO 命令中的某些统计数据，包括：

* Keyspace hits (键空间命中次数)
* Keyspace misses (键空间不命中次数)
* Number of commands processed (执行命令的次数)
* Number of connections received (连接服务器的次数)
* Number of expired keys (过期key的数量)
* Number of rejected connections (被拒绝的连接数量)
* Latest fork(2) time(最后执行 fork(2) 的时间)
* The aof_delayed_fsync counter(aof_delayed_fsync 计数器的值)

### CONFIG REWRITE
CONFIG REWRITE 命令对启动 Redis 服务器时所指定的 redis.conf 文件进行改写： 因为 CONFIG SET 命令可以对服务器的当前配置进行修改， 而修改后的配置可能和 redis.conf 文件中所描述的配置不一样， CONFIG REWRITE 的作用就是通过尽可能少的修改， 将服务器当前所使用的配置记录到 redis.conf 文件中。

重写会以非常保守的方式进行：

* 原有 redis.conf 文件的整体结构和注释会被尽可能地保留。
* 如果一个选项已经存在于原有 redis.conf 文件中 ， 那么对该选项的重写会在选项原本所在的位置（行号）上进行。
* 如果一个选项不存在于原有 redis.conf 文件中， 并且该选项被设置为默认值， 那么重写程序不会将这个选项添加到重写后的 redis.conf 文件中。
* 如果一个选项不存在于原有 redis.conf 文件中， 并且该选项被设置为非默认值， 那么这个选项将被添加到重写后的 redis.conf 文件的末尾。
* 未使用的行会被留白。 比如说， 如果你在原有 redis.conf 文件上设置了数个关于 save 选项的参数， 但现在你将这些 save 参数的一个或全部都关闭了， 那么这些不再使用的参数原本所在的行就会变成空白的。
* 即使启动服务器时所指定的 redis.conf 文件已经不再存在， CONFIG REWRITE 命令也可以重新构建并生成出一个新的 redis.conf 文件。

另一方面， 如果启动服务器时没有载入 redis.conf 文件或者没有指定配置文件或者配置文件写权限不够， 那么执行 CONFIG REWRITE 命令将引发一个错误。

对 redis.conf 文件的重写是原子性的， 并且是一致的： 如果重写出错或重写期间服务器崩溃， 那么重写失败， 原有 redis.conf 文件不会被修改。 如果重写成功， 那么 redis.conf 文件为重写后的新文件。


### DBSIZE
* 返回当前数据库的 key 的数量。

### DEBUG OBJECT
* DEBUG OBJECT key

DEBUG OBJECT 是一个调试命令，它不应被客户端所使用。

查看 OBJECT 命令获取更多信息。


### DEBUG SEGFAULT
* DEBUG SEGFAULT

执行一个不合法的内存访问从而让 Redis 崩溃，仅在开发时用于 BUG 模拟。

### FLUSHDB
清空当前数据库中的所有 key。异步执行。

此命令从不失败。

### FLUSHALL
清空整个 Redis 服务器的数据(删除所有数据库的所有 key )。异步执行。

此命令从不失败。

### INFO
* INFO [section]

以一种易于解释（parse）且易于阅读的格式，返回关于 Redis 服务器的各种信息和统计数值。

通过给定可选的参数 section ，可以让命令只返回某一部分的信息：

server : 一般 Redis 服务器信息，包含以下域：

* redis_version : Redis 服务器版本
* redis_git_sha1 : Git SHA1
* redis_git_dirty : Git dirty flag
* os : Redis 服务器的宿主操作系统
* arch_bits : 架构（32 或 64 位）
* multiplexing_api : Redis 所使用的事件处理机制
* gcc_version : 编译 Redis 时所使用的 GCC 版本
* process_id : 服务器进程的 PID
* run_id : Redis 服务器的随机标识符（用于 Sentinel 和集群）
* tcp_port : TCP/IP 监听端口
* uptime_in_seconds : 自 Redis 服务器启动以来，经过的秒数
* uptime_in_days : 自 Redis 服务器启动以来，经过的天数
* lru_clock : 以分钟为单位进行自增的时钟，用于 LRU 管理


clients : 已连接客户端信息，包含以下域：

* connected_clients : 已连接客户端的数量（不包括通过从属服务器连接的客户端）
* client_longest_output_list : 当前连接的客户端当中，最长的输出列表
* client_longest_input_buf : 当前连接的客户端当中，最大输入缓存
* blocked_clients : 正在等待阻塞命令（BLPOP、BRPOP、BRPOPLPUSH）的客户端的数量


memory : 内存信息，包含以下域：

* used_memory : 由 Redis 分配器分配的内存总量，以字节（byte）为单位
* used_memory_human : 以人类可读的格式返回 Redis 分配的内存总量
* used_memory_rss : 从操作系统的角度，返回 Redis 已分配的内存总量（俗称常驻集大小）。这个值和 top 、 ps 等命令的输出一致。
* used_memory_peak : Redis 的内存消耗峰值（以字节为单位）
* used_memory_peak_human : 以人类可读的格式返回 Redis 的内存消耗峰值
* used_memory_lua : Lua 引擎所使用的内存大小（以字节为单位）
* mem_fragmentation_ratio : used_memory_rss 和 used_memory 之间的比率
* mem_allocator : 在编译时指定的， Redis 所使用的内存分配器。可以是 libc 、 jemalloc 或者 tcmalloc 。

在理想情况下， used_memory_rss 的值应该只比 used_memory 稍微高一点儿。
当 rss > used ，且两者的值相差较大时，表示存在（内部或外部的）内存碎片。
内存碎片的比率可以通过 mem_fragmentation_ratio 的值看出。
当 used > rss 时，表示 Redis 的部分内存被操作系统换出到交换空间了，在这种情况下，操作可能会产生明显的延迟。

当 Redis 释放内存时，分配器可能会，也可能不会，将内存返还给操作系统。
如果 Redis 释放了内存，却没有将内存返还给操作系统，那么 used_memory 的值可能和操作系统显示的 Redis 内存占用并不一致。
查看 used_memory_peak 的值可以验证这种情况是否发生。

persistence : RDB 和 AOF 的相关信息

stats : 一般统计信息

replication : 主/从复制信息

cpu : CPU 计算量统计信息

commandstats : Redis 命令统计信息

cluster : Redis 集群信息

keyspace : 数据库相关的统计信息

除上面给出的这些值以外，参数还可以是下面这两个：
* all : 返回所有信息
* default : 返回默认选择的信息
* 当不带参数直接调用 INFO 命令时，使用 default 作为默认参数。


### LASTSAVE
返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示。

### MONITOR
实时打印出 Redis 服务器接收到的命令，调试用。

```
127.0.0.1:6379> MONITOR
OK
# 以第一个打印值为例
# 1378822099.421623 是时间戳
# [0 127.0.0.1:56604] 中的 0 是数据库号码， 127... 是 IP 地址和端口
1378822105.089572 [0 127.0.0.1:56604] "SET" "msg" "hello world"
```

### PSYNC/SYNC
*  用于复制功能(replication)的内部命令

 2.8 或以上版本 ->  PSYNC
 2.8 以下 ->  SYNC
 
###  SAVE
* SAVE 命令执行一个同步保存操作，将当前 Redis 实例的所有数据快照(snapshot)以 RDB 文件的形式保存到硬盘。

一般来说，在生产环境很少执行 SAVE 操作，因为它会阻塞所有客户端，保存数据库的任务通常由 BGSAVE 命令异步地执行。然而，如果负责保存数据的后台子进程不幸出现问题时， SAVE 可以作为保存数据的最后手段来使用。

### SHUTDOWN
SHUTDOWN 命令执行以下操作：

* 停止所有客户端
* 如果有至少一个保存点在等待，执行 SAVE 命令
* 如果 AOF 选项被打开，更新 AOF 文件
* 关闭 redis 服务器(server)
* 如果持久化被打开的话， SHUTDOWN 命令会保证服务器正常关闭而不丢失任何数据。

另一方面，假如只是单纯地执行 SAVE 命令，然后再执行 QUIT 命令，则没有这一保证 —— 因为在执行 SAVE 之后、执行 QUIT 之前的这段时间中间，其他客户端可能正在和服务器进行通讯，这时如果执行 QUIT 就会造成数据丢失。

SAVE 和 NOSAVE 修饰符

通过使用可选的修饰符，可以修改 SHUTDOWN 命令的表现。比如说：

* 执行 SHUTDOWN SAVE 会强制让数据库执行保存操作，即使没有设定(configure)保存点
* 执行 SHUTDOWN NOSAVE 会阻止数据库执行保存操作，即使已经设定有一个或多个保存点(你可以将这一用法看作是强制停止服务器的一个假想的 ABORT 命令)

### SLOWLOG
* SLOWLOG subcommand [argument]

Slow log 是 Redis 用来记录查询执行时间的日志系统。

查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间。

另外，slow log 保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启 slow log 而损害 Redis 的速度。

设置 SLOWLOG
* 第一个选项是 slowlog-log-slower-than ，它决定要对执行时间大于多少微秒(microsecond，1秒 = 1,000,000 微秒)的查询进行记录。

* 另一个选项是 slowlog-max-len ，它决定 slow log 最多能保存多少条日志， slow log 本身是一个 FIFO 队列，当队列大小超过 slowlog-max-len 时，最旧的一条日志将被删除，而最新的一条日志加入到 slow log ，以此类推。


查看 slow log
* SLOWLOG GET  打印所有 slow log ，最大长度取决于 slowlog-max-len 选项的值
* SLOWLOG GET number 只打印指定数量的日志。

最新的日志会最先被打印：
``` 
redis> SLOWLOG GET
1) 1) (integer) 12                      # 唯一性(unique)的日志标识符
   2) (integer) 1324097834              # 被记录命令的执行时间点，以 UNIX 时间戳格式表示
   3) (integer) 16                      # 查询执行时间，以微秒为单位
   4) 1) "CONFIG"                       # 执行的命令，以数组的形式排列
      2) "GET"                          # 这里完整的命令是 CONFIG GET slowlog-log-slower-than
      3) "slowlog-log-slower-than"

```
  
日志的唯一 id 只有在 Redis 服务器重启的时候才会重置，这样可以避免对日志的重复处理(比如你可能会想在每次发现新的慢查询时发邮件通知你)。

查看当前日志的数量：SLOWLOG LEN 
清空日志:SLOWLOG RESET

### SLAVEOF
* SLAVEOF host port /  SLAVEOF NO ONE
* SLAVEOF 命令用于在 Redis 运行时动态地修改复制(replication)功能的行为。

通过执行 SLAVEOF host port 命令，可以将当前服务器转变为指定服务器的从属服务器(slave server)。

如果当前服务器已经是某个主服务器(master server)的从属服务器，那么执行 SLAVEOF host port 将使当前服务器停止对旧主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。

另外，对一个从属服务器执行命令 SLAVEOF NO ONE 将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得的数据集不会被丢弃。

利用『 SLAVEOF NO ONE 不会丢弃同步所得数据集』这个特性，可以在主服务器失败的时候，将从属服务器用作新的主服务器，从而实现无间断运行。


### TIME
* 返回当前服务器时间。

 
 
 
 
 










