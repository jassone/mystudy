## InnoDB的内存架构
InnoDB的内存架构主要分为三大块，缓冲池（Buffer Pool）、重做缓冲池（Redo Log Buffer）和额外内存池。

![20220211171520.jpg](https://pic.imgdb.cn/item/620629412ab3f51d916984c2.jpg)

## 一、缓冲池（Buffer Pool）
mysql中数据是存储在物理磁盘上的，而真正的数据处理又是在内存中执行的。由于磁盘的读写速度非常慢，如果每次操作都对磁盘进行频繁读写的话，那么性能一定非常差。为了上述问题，InnoDB将数据划分为若干页，以页作为磁盘与内存交互的基本单位，一般页的大小为16KB。这样的话，一次性至少读取1页数据到内存中或者将1页数据写入磁盘。通过减少内存与磁盘的交互次数，从而提升性能。

其实，这**本质上就是一种典型的缓存设计思想**，一般缓存的设计基本都是从时间维度或者空间维度进行考量的：
* 时间维度：如果一条数据正在在被使用，那么在接下来一段时间内大概率还会再被使用。可以认为热点数据缓存都属于这种思路的实现。
* 空间维度：如果一条数据正在在被使用，那么存储在它附近的数据大概率也会很快被使用。InnoDB的数据页和操作系统的页缓存则是这种思路的体现。

**为了缓存磁盘中的页**，在 MySQL 服务器启动的时候就向操作系统申请了一片连续的内存，叫做 Buffer Pool 。Buffer Pool是MySQL或者说InnoDB中，十分重要、核心的一部分，位于主存。**在真正访问数据页之前，需要先把磁盘中的页加载到内存中的Buffer Pool中，这就是为什么其访问数据的效率很高。**

### 1、特性
通常来说，宿主机80%的内存都应该分配给Buffer Pool，因为Buffer Pool越大，其能缓存的数据就更多，更多的操作都会发生在内存，从而达到提升效率的目的。

由于其存储的数据类型和数据量非常多，Buffer Pool存储的时候一定会按照某些结构去存储，并且做了某些处理。否则获取的时候除了遍历所有数据之外，没有其他的捷径，这样的低效率操作肯定是无法支撑MySQL的高性能的。

因此，Buffer Pool被分成了很多页。每页可以存放很多数据，刚刚也提到了，InnoDB一定是对数据做了某些操作。

InnoDB使用了链表来组织页和页中存储的数据，页与页之间形成了双向链表，这样可以方便的从当前页跳到下一页，同时使用LRU（Least Recently Used）算法去淘汰那些不经常使用的数据。

![20220122121649.jpg](https://pic.imgdb.cn/item/61eb85392ab3f51d91a63154.jpg)

同时，每页中的数据也通过单向链表进行链接。因为这些数据是分散到Buffer Pool中的，单向链表将这些分散的内存给连接了起来。
![20220122122928.jpg](https://pic.imgdb.cn/item/61eb886f2ab3f51d91a8fd3b.jpg)

### 2、Buffer Pool的LRU算法
Buffer Pool对应的内存大小是有限的，如果使用完了，就要把某些就的缓冲页从Buffer Pool中移除。那么哪些缓冲页该移除呢？

先说下降低Buffer Pool的命中率的两种情况有：
* 加载到Buffer Pool中的页不一定被用到。
* 如果有非常多的使用频率低的页被同时加载到Buffer Pool中，可能会把那些使用频率高的页从Buffer Pool中淘礼。

##### 原生LRU
首先明确一点，此处的LRU算法和我们传统的LRU算法有一定的区别。为什么呢？因为实际生产环境中会存在全表扫描的情况，如果数据量较大，可能会将Buffer Pool中存下来的热点数据给全部替换出去，而这样就会导致该段时间MySQL性能断崖式下跌。

对于这种情况，MySQL有一个专用名词叫`缓冲池污染`。所以MySQL对LRU算法做了优化。

##### 优化后的LRU
优化之后的链表被分成了两个部分，分别是 New Sublist 和 Old Sublist，其分别占用了 Buffer Pool 的3/4和1/4。
![20220213085205.jpg](https://pic.imgdb.cn/item/620856422ab3f51d91293767.jpg)

链表的前3/4，也就是 New Sublist 存放的是访问较为频繁的页，而后1/4也就是 Old Sublist 则是访问的不那么频繁的页。Old Sublist中的数据，会在后续Buffer Pool剩余空间不足、或者有新的页加入时被移除掉。

该链表存储的数据来源有两部分，分别是：
* MySQL的`预读`线程预先加载的数据
* 用户的操作，例如Query查询(比如全表扫描)

预读的其他页数据或全表扫描页数据，这些页数据会被放在Old Sublist的头部区域，后期有再次访问这些页数据，则会移到New Sublist区域，否则后面会被慢慢淘汰掉。

![20220213085532.jpg](https://pic.imgdb.cn/item/6208570a2ab3f51d9129bd62.jpg)

### 3、自适应哈希索引
自适应哈希索引（Adaptive Hash Index）是配合Buffer Pool工作的一个功能。自适应哈希索引使得MySQL的性能更加接近于内存服务器。

详见`自适应哈希索引`那篇文章。

### 4、刷新脏页到磁盘
如果我们修改了Buffer Pool中某个缓冲页的数据，它就与磁盘上的页不一样了，这样的缓冲页称为脏页。

执行时机：
* redo log写满了
* 系统内存缓存区不足了，就要淘汰一些数据页，如果淘汰的是脏页数据，就要先将脏页写到磁盘。
* mysql比较空闲的时候。
* msyql正常shotdown时，会先把内存脏页都flush到磁盘。
* 后台有个专门的线程负责每隔一段时间就把脏页数据刷新到磁盘。

### 5、二次写
##### 数据库宕机导致数据丢失
如果运行中的MySQL数据库突然发生宕机，此时InnoDB存储引擎正在将缓冲池中的页持久化到磁盘中，但是这个时刻数据库宕机了，那么会导致部分数据刷到了磁盘中，部分数据还在内存中，这种情况在MySQL中称为partial page write
(部分写失效)。

##### redo log无法对页损坏做恢复
redo log记录的是对页的物理操作，例如:偏移量800,写入'xxx'记录，但是页本身如果是损坏的，那么redo也是无济于事的。

##### 原理和流程
![727246-20200922172156139-1080371140.jpeg](https://pic.imgdb.cn/item/6208e2f12ab3f51d91a5bdd7.jpg)
double write由两部分组成，一部分是内存中的double write buffer, 其大小未2M;另一部分为磁盘上 ibdata中连续的128个页，其大小也是2M。

1. 当一系列机制触发缓冲池中脏页刷新时，并不直接写入磁盘数据文件中，而是先拷贝至内存中的double write buffer中。

2. 接着从两次写缓冲区中分两次写入磁盘共享表空间中(连续存储，顺序写，性能很高)，每次写1M

3. 待第二部完成之后，再将double write buffer中的脏页数据写入实际的各个表空间文件中(离散写)，脏页数据固化后，即进行标记对应的double write数据可覆盖。

这样在宕机重启时，如果出现数据页损坏，那么在应用redo log之前，需要通过该页的副本来还原该页，然后再进行redo log重做，这就是double write。

### 6、优化Buffer Pool的配置
在实际的生产环境中，我们可以通过变更某些设置，来提升Buffer Pool运行的性能。
* 例如，我们可以分配尽量多的内存给Buffer Pool，如此就可以缓存更多的数据在内存中
* 当前有足够的内存时，就可以搞多个Buffer Pool实例，减少并发操作所带来的数据竞争
* 当我们可以预测到即将到来的大量请求时，我们可以手动的执行这部分数据的预读请求
* 我们还可以控制Buffer Pool刷数据到磁盘的频率，以根据当前MySQL的负载动态调整

那我们怎么知道当前运行的 MySQL 中 Buffer Pool 的状态呢？我们可以通过命令**show engine innodb status**来查看。这个命令是看 InnoDB 整体的状态的， Buffer Pool 相关的监控指标包含在了其中，在Buffer Pool And Memory模块中。

```sh
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 137428992
Dictionary memory allocated 972752
Buffer pool size   8191
Free buffers       4596
Database pages     3585
Old database pages 1303
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1171, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 655, created 7139, written 173255
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 3585, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
```

解释一些关键的指标所代表的含义：
* Total memory allocated：分配给 Buffer Pool 的总内存
* Dictionary memory allocated：分配给 InnoDB 数据字典的总内存
* Buffer pool size：分配给 Buffer Pool 中页的内存大小
* Free buffers：分配给 Buffer Pool 中 Free List 的内存大小
* Database pages：分配给 LRU 链表的内存大小
* Old database pages：分配给 LRU 子链表的内存大小
* Modified db pages：当前Buffer Pook中被更新的页的数量
* Pending reads：当前等待读入 Buffer Pool 的页的数量
* Pending writes LRU：当前在 LRU 链表中等待被刷入磁盘的脏页数量

都是些很常规的配置项，你可能会比较好奇什么是 Free List。

Free List 中存放的都是未被使用的页。因为MySQL启动的时候，InnoDB 会预先申请一部分页。如果当前页还未被使用，就会被保存在 Free List 中。

知道了 Free List，那么你也应该知道 Flush List，里面保存的是所有的脏页，都是被更改后需要刷入到磁盘的。

## 二、Log Buffer
Log Buffer用来存储那些即将被刷入到磁盘文件中的日志，例如Redo Log，该区域也是InnoDB内存的重要组成部分。Log Buffer的默认值为16M，如果我们需要进行调整的话，可以通过配置参数innodb_log_buffer_size来进行调整。

当Log Buffer如果较大，就可以存储更多的Redo Log，这样一来在事务提交之前我们就不需要将Redo Log刷入磁盘，只需要丢到Log Buffer中去即可。因此较大的Log Buffer就可以更好的支持较大的事务运行；同理，如果有事务会大量的更新、插入或者删除行，那么适当的增大Log Buffer的大小，也可以有效的减少部分磁盘I/O操作。

至于Log Buffer中的数据刷入到磁盘的频率，则可以通过参数innodb_flush_log_at_trx_commit来决定。
* 设置为 0 时，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中；
* 设置为 1 时，表示每次事务提交时都将 redo log 直接持久化到磁盘；
* 设置为 2 时，表示每次事务提交时都只是把 redo log 写到 page cache。

InnoDB 有一个后台线程，每隔 1 秒，就会把 redo log buffer 中的日志，调用 write 写到文件系统的 page cache，然后调用 fsync 持久化到磁盘。

## 三、Change Buffer
聊完了 Buffer Pool 中索引相关，剩下的就是 Change Buffer 了。Change Buffer是一块比较特殊的区域，其作用是用于存储那些当前不在 Buffer Pool 中的但是又被修改过的二级索引。

用流程来描述一下就是，当我们更新了非聚簇索引（二级索引）的数据时，此时应该是直接将其在Buffer Pool中的对应数据更新了即可，但是不凑巧的是，当前二级索引不在 Buffer Pool 中，此时将其从磁盘拉取到 Buffer Pool 中的话，并不是最优的解，因为该二级索引可能之后根本就不会被用到，那么刚刚昂贵的磁盘I/O操作就白费了。

所以，我们需要这么一个地方，来暂存对这些二级索引所做的改动。当被缓存的二级索引页被其他的请求加载到了Buffer Pool 中之后，就会将 Change Buffer 中缓存的数据合并到 Buffer Pool 中去。

当然，Change Buffer也不是没有缺点。当 Change Buffer 中有很多的数据时，全部合并到Buffer Pool可能会花上几个小时的时间，并且在合并的期间，磁盘的I/O操作会比较频繁，从而导致部分的CPU资源被占用。

那你可能会问，难道只有被缓存的页加载到了 Buffer Pool 才会触发合并操作吗？那要是它一直没有被加载进来，Change Buffer 不就被撑爆了？很显然，InnoDB在设计的时候考虑到了这个点。除了对应的页加载，提交事务、服务停机、服务重启都会触发合并。

## 4、相关wiki
* https://mp.weixin.qq.com/s?__biz=MzU5NDk0MTc2OA==&mid=2247484531&idx=1&sn=947f880c4b4d68a297ea150a1497a561&scene=21#wechat_redirect

* https://zhuanlan.zhihu.com/p/400947554
