### LINDEX
* LINDEX key index
* index=0 表示第一个元素，负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素
* O(N)， N 为到达下标 index 过程中经过的元素数量。因此，对列表的头元素和尾元素执行 LINDEX 命令，复杂度为**O(1)**。
* 返回：列表中下标为 index 的元素。如果 index 参数的值不在列表的区间范围内(out of range)，返回 nil 

### LINSERT
* LINSERT key BEFORE|AFTER pivot value
* 将值 value 插入到列表 key 当中，位于值 pivot 之前或之后
* 当 pivot 不存在于列表key,或者当 key 不存在时， key 被视为空列表，不执行任何操作
* O(N)， N 为寻找 pivot 过程中经过的元素数量。
* 返回：如果命令执行成功，返回插入操作完成之后，列表的长度。如果没有找到 pivot ，返回 -1 。如果 key 不存在或为空列表，返回 0 。


### LLEN

### LPOP
* 移除并返回列表 key 的头元素。

### LPUSH
* LPUSH key value [value ...]
* **O(1)**
* 原子性操作执行完所有动作
* 返回：执行 LPUSH 命令后，列表的长度

### LPUSHX
* LPUSHX key value
* 将值 value 插入到列表 key 的表头，当且仅当 key 存在并且是一个列表。
* key 不存在时，不处理
* 返回：命令执行之后，表的长度

### LRANGE
* LRANGE key start stop
* 返回：O(S+N)， S 为偏移量 start ， N 为指定区间内元素的数量

###### 超出范围的下标(不会引起错误)
* 如果 start 下标比列表的最大下标 end ( LLEN list 减去 1 )还要大，那么 LRANGE 返回一个空列表。
* 如果 stop 下标比 end 下标还要大，Redis将 stop 的值设置为 end 。

### LREM
* LREM key count value
* 根据参数 count 的值，移除列表中与参数 value 相等的元素
* count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，**数量为 count**
* count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，**数量为 count 的绝对值**
* count = 0 : 移除表中所有与 value 相等的值
* O(N)， N 为列表的长度
* 返回：被移除元素的数量

### LSET
* LSET key index value
* 对头元素或尾元素进行 LSET 操作，复杂度为 O(1)。其他情况下，为 O(N)， N 为列表的长度。
* <font color="red">当 index 参数超出范围，或对一个空列表( key 不存在)进行 LSET 时，返回一个错误</font>

### LTRIM
* LTRIM key start stop
* O(N)， N 为被移除的元素的数量。
* 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
* 超出范围的下标值不会引起错误。
1 如果 start 下标比列表的最大下标 end ( LLEN list 减去 1 )还要大，或者 start > stop ， LTRIM 返回一个空列表(因为 LTRIM 已经将整个列表清空)。2 如果 stop 下标比 end 下标还要大，Redis将 stop 的值设置为 end 。


##### 应用：LTRIM 命令通常和 LPUSH 命令或 RPUSH 命令配合使用，举个例子：
>     LPUSH log newest_log
>     LTRIM log 0 99
这个例子模拟了一个日志程序，每次将最新日志 newest_log 放到 log 列表中，并且只保留最新的 100 项。注意当这样使用 LTRIM 命令时，时间复杂度是O(1)，因为平均情况下，每次只有一个元素被移除。

### RPOP
* 移除并返回列表 key 的尾元素。

### RPUSH
* RPUSH key value [value ...]
* O(1)
* 返回:执行 RPUSH 操作后，表的长度
* 如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表尾

### RPUSHX
* RPUSHX key value
* 返回:RPUSHX 命令执行之后，表的长度。

### BRPOP
* BRPOP key [key ...] timeout
* O(1)
* 是 RPOP 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BRPOP 命令阻塞，直到等待超时或发现可弹出元素为止
* 超时参数 timeout 接受一个以秒为单位的数字作为值。超时参数设为 0 表示阻塞时间可以无限期延长(block indefinitely) 
* 当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的尾部元素
* 返回：假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值
* <font color="red">注意：如果设置的超时时间太长，这个连接太久没有活跃过，可能会被 Redis Server 判定为无效连接，之后 Redis Server 会强制把这个客户端踢下线。所以，采用这种方案，客户端要有重连机制。</font>

### BLPOP（同BRPOP）
* BLPOP key [key ...] timeout

##### 模式
* 有时候，为了等待一个新元素到达数据中，需要使用轮询的方式对数据进行探查。另一种更好的方式是，使用系统提供的阻塞原语，在新元素到达时立即进行处理，而新元素还没到达时，就一直阻塞住，避免轮询占用资源。

##### 说明
相同的 key 可以被多个客户端同时阻塞。
* 不同的客户端被放进一个队列中，按『先阻塞先服务』(first-BLPOP，first-served)的顺序为 key 执行 BLPOP 命令。

在MULTI/EXEC事务中的BLPOP
* BLPOP 可以用于流水线(pipline,批量地发送多个命令并读入多个回复)，但把它用在 MULTI / EXEC 块当中没有意义。因为这要求整个服务器被阻塞以保证块执行时的原子性，该行为阻止了其他客户端执行 LPUSH 或 RPUSH 命令。
* 因此，一个被包裹在 MULTI / EXEC 块内的 BLPOP 命令，行为表现得就像 LPOP 一样，对空列表返回 nil ，对非空列表弹出列表元素，不进行任何阻塞操作。

### RPOPLPUSH
* RPOPLPUSH source destination
* O(1)
* 如果 source 和 destination 相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作<font color="red">列表的旋转(rotation)操作</font>

##### 命令 RPOPLPUSH 在一个原子时间内，执行以下两个动作：
* 将列表 source 中的最后一个元素(尾元素)弹出，并返回给客户端。
* 将 source 弹出的元素插入到列表 destination ，作为 destination 列表的的头元素

##### 模式-循环列表
* 通过使用相同的 key 作为 RPOPLPUSH 命令的两个参数，客户端可以用一个接一个地获取列表元素的方式，取得列表的所有元素，而不必像 LRANGE 命令那样一下子将所有列表元素都从服务器传送到客户端中(两种方式的总复杂度都是 O(N))。

以上的模式甚至在以下的两个情况下也能正常工作：

* 有多个客户端同时对同一个列表进行旋转(rotating)，它们获取不同的元素，直到所有元素都被读取完，之后又从头开始。
* 有客户端在向列表尾部(右边)添加新元素。

这个模式使得我们可以很容易实现这样一类系统：有 N 个客户端，需要连续不断地对一些元素进行处理，而且处理的过程必须尽可能地快。一个典型的例子就是服务器的监控程序：它们需要在尽可能短的时间内，并行地检查一组网站，确保它们的可访问性。

注意，使用这个模式的客户端是易于扩展(scala)且安全(reliable)的，因为就算接收到元素的客户端失败，元素还是保存在列表里面，不会丢失，等到下个迭代来临的时候，别的客户端又可以继续处理这些元素了。

### BRPOPLPUSH(RPOPLPUSH的阻塞版本)
* BRPOPLPUSH source destination timeout

##### 模式-安全的队列
* 一个客户端可能在取出一个消息之后崩溃，而未处理完的消息也就因此丢失。
* 使用 RPOPLPUSH 命令(或者它的阻塞版本 BRPOPLPUSH )可以解决这个问题：因为它不仅返回一个消息，同时还将这个消息添加到另一个备份列表当中，如果一切正常的话，当一个客户端完成某个消息的处理之后，可以用 LREM 命令将这个消息从备份表删除。
* 最后，还可以添加一个客户端专门用于监视备份表，它自动地将超过一定处理时限的消息重新放入队列中去(负责处理该消息的客户端可能已经崩溃)，这样就不会丢失任何消息了。



