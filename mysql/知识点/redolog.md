## redo log

redo log叫做重做日志，是保证事务持久性的重要机制。当mysql服务器意外崩溃或者宕机后，保证已经提交的事务，确定持久化到磁盘中的一种措施。

### 1、为什么需要redo log？

innodb是以页为单位来管理存储空间的，任何的增删改查操作最终都会操作完整的一个页，会将整个页加载到buffer pool中，然后对需要修改的记录进行修改，修改完毕不会立即刷新到磁盘，原因是：

- 仅仅修改了一条记录，刷新一个完整的数据页的话过于浪费了。
- 一个事务会涉及不同的页面，如果将这些页面都刷盘会产生很多的随机IO。

但是如果不立即刷新的话，数据此时还在内存中，如果此时发生系统崩溃最终数据会丢失的，因此权衡利弊，引入了redo log，也就是说，修改完后不立即刷新，对任意页面进行修改的操作都会生成redo日志，**在事务提交时**，只要保证生成的redo日志成功落盘即可。而且记录一条日志，日志内容就是记录哪个页面，多少偏移量，什么数据发生了什么变更。这样即使系统崩溃，再恢复后，也可以根据redo日志进行数据恢复。

这样，即使MySQL发生故障导致内存中的数据丢失，也可以根据已落盘的redo日志恢复数据。

### 2、使用redo日志的好处

- **redo日志记录的是每个页更改的物理情况**，占用的空间小

- 一个事务生成的redo日志是按顺序写入磁盘的，是顺序IO

## 一、黑盒下的更新数据流程

当我们查询数据的时候，会先去Buffer Pool中查询。如果Buffer Pool中不存在，存储引擎会先将数据从磁盘加载到Buffer Pool中，然后将数据返回给客户端；同理，当我们更新某个数据的时候，如果这个数据不存在于Buffer Pool，同样会先数据加载进来，然后修改修改内存的数据。被修改过的数据会在之后统一刷入磁盘。

![20220213174216.jpg](https://pic.imgdb.cn/item/6208d2fc2ab3f51d91974f3d.jpg)

这个过程看似没啥问题，实则不讲武德。假设我们修改Buffer Pool中的数据成功，但是还没来得及将数据刷入磁盘MySQL就挂了怎么办？按照上图的逻辑，此时更新之后的数据只存在于Buffer Pool中，如果此时MySQL宕机了，这部分数据将会永久的丢失；

再者，我更新到一半突然发生错误了，想要回滚到更新之前的版本，该怎么办？那不完犊子吗，连数据持久化的保证、事务回滚都做不到还谈什么崩溃恢复？

## 二、Redo Log & Undo Log
而通过MySQL能够实现崩溃恢复的事实来看，MySQL必定实现了某些骚操作。没错，这就是接下来我们要介绍的另外的两个关键功能，Redo Log和Undo Log。

这两种日志是属于InnoDB存储引擎的日志，和MySQL Server的Binlog不是一个维度的日志。

* Redo Log 记录了此次事务「完成后」的数据状态，记录的是更新之「后」的值
* Undo Log 记录了此次事务「开始前」的数据状态，记录的是更新之「前」的值

## 三、实现日志后的更新流程

**先写undolog，再写redolog。**

![20220213174415.jpg](https://pic.imgdb.cn/item/6208d3172ab3f51d91976728.jpg)

1、在更新Buffer Pool中的数据之前，我们需要先将该数据事务开始之前的状态写入Undo Log中。假设更新到一半出错了，我们就可以通过Undo Log来回滚到事务开始前。

2、然后执行器会更新Buffer Pool中的数据，成功更新后会将数据最新状态写入Redo Log Buffer中。因为一个事务中可能涉及到多次读写操作，写入Buffer中分组写入，比起一条条的写入磁盘文件，效率会高很多。最后再刷到redolog中。

![20220213174814.jpg](https://pic.imgdb.cn/item/6208d3e62ab3f51d91981fdf.jpg)

那为什么Undo Log不也搞一个Undo Log Buffer，也给Undo Log提提速，雨露均沾？那我们假设有这个一个Buffer存在于InnoDB，将事务开始前的数据状态写入了Undo Log Buffer中，然后开始更新数据。

突然啪一下，很快啊，MySQL由于意外进程退出了，此时会发生一件很尴尬的事情，如果更新的数据一部分已经刷回磁盘了，但是此时事务没有成功需要回滚，你发现Undo Log随着进程退出一起没了，此时就没有办法通过Undo Log去做回滚。

那如果刚刚更新完内存，MySQL就挂了呢？此时Redo Log Buffer甚至都可能没有写入，即使写入了也没有刷到磁盘，Redo Log也丢了。

其实无所谓，因为意外宕机，该事务没有成功，既然事务事务没有成功那就需要回滚，而MySQL重启后会读取磁盘上的Redo Log文件，将其状态给加载到Buffer Pool中。而通过磁盘Redo Log文件恢复的状态和宕机前事务开始前的状态是一样的，所以是没有影响的。然后等待事务commit了之后就会将Redo Log和Binlog刷到磁盘。

## 四、redolog 2PC（两阶段提交）
### 1、当同时在写binlog时存在的问题
因为InnoDB和执行器是两套系统(**redo日志和undo日志是Innodb层的，而binlog是server层的**)，所以日志必须要保持一致，否则会出现问题。

##### a、先写redo log后写binlog：
* 假设在redo log写完，binlog还没写完时，Mysql进程异常重启。这时候，可以依靠redo log将数据恢复回来，假设恢复后这行数据自动c=1。
* 由于binlog没写完，这时候binlog里面就没有记录这个语句。 如果需要用这个binlog来恢复临时库的话，由于这个语句的binlog丢失，这个临时库就会少了这一次更新，恢复出来的行数据值c=0，与原库不同。

![03dfe87c59.png](https://pic.imgdb.cn/item/63f32ae4f144a010076d076c.png)

###### b、先写binlog后写redo log：

* 假设在binlog写完后宕机，由于redo log还没写，重启mysql后这个事务无效，所以这行数据字段c=0。
* 但是binlog里面已经记录了“把c改为1”这个日志，所以之后用binlog来恢复的时候就多了一个事务出来，恢复出来的这行字段c=1,与原库不同。

![8cb2c4419e3.png](https://pic.imgdb.cn/item/63f32b1df144a010076d5103.png)

### 2、基于2PC的一致性保障

从这你可以发现一个关键的问题，那就是必须保证Redo Log和Binlog在事务提交时的数据一致性，要么都存在，要么都不存在。MySQL是通过 2PC（two-phase commit protocol）来实现的。

![20220213175528.jpg](https://pic.imgdb.cn/item/6208d5972ab3f51d919a5b96.jpg)

简单介绍一下2PC，它是一种保证分布式事务数据一致性的协议，它中文名叫两阶段提交，它将分布式事务的提交拆分成了2个阶段，分别是Prepare和Commit/Rollback。

就像两个拳击手开始比赛之前，裁判会在中间确认两个选手的状态，类似于问你准备好了吗？得到确认之后，裁判才会说Fight。

裁判询问选手的状态，对应的是第一阶段Prepare；得到了肯定的回答之后，裁判宣布比赛正式开始，对应的是第二阶段Commit，但是如果有一方选手没有准备好，裁判会宣布比赛暂停，此时对应的是第一阶段失败的情况，第二阶段的状态会变为Rollback。裁判就对应2PC中的协调者Coordinator，选手就对应参与者Participant。

下面我们通过一张图来看一下整个流程。
![20220213175641.jpg](https://pic.imgdb.cn/item/6208d5e12ab3f51d919ab04c.jpg)
Prepare阶段，将Redo Log写入文件，并刷入磁盘，记录上内部XA事务的ID，同时将Redo Log状态设置为Prepare。Redo Log写入成功后，再将Binlog同样刷入磁盘，记录XA事务ID。

Commit阶段，向磁盘中的Redo Log写入Commit标识，表示事务提交。然后执行器调用存储引擎的接口提交事务。这就是整个过程。

##### 验证2PC机制的可用性
* 假设Redo Log刷入成功了，但是还没来得及刷入Binlog MySQL就挂了。此时重启之后会发现Redo Log并没有Commit标识，此时根据记录的XA事务找到这个事务，进行回滚。

* 如果Redo Log刷入成功，而且Binlog也刷入成功了，但是还没有来的及将Redo Log从Prepare改成Commit MySQL就挂了，此时重启会发现虽然Redo Log没有Commit标识，但是通过XID查询到的Binlog却已经成功刷入磁盘了。此时，虽然Redo Log没有Commit标识，MySQL也要提交这个事务。因为Binlog一旦写入，就可能会被从库或者任何消费Binlog的消费者给消费。如果此时MySQL不提交事务，则可能造成数据不一致。而且目前Redo Log和Binlog从数据层面上，其实已经Ready了，只是差个标志位。

![5dcadaea56.png](https://pic.imgdb.cn/item/63f32b3af144a010076d7371.png)

## 五、redo日志刷盘时机
* log buffer 空间不足时
* 事务提交时(持久性的保证)
* 将某个脏页数据刷新到磁盘前，会保证先将该脏页对应的redo日志刷新到磁盘中(redo日志是顺序存放的，所以会保证将该日志之前的产生的redo日志也刷新到磁盘)。
* 后台线程不停的刷，大约每秒都会刷新一次
* 正常关闭服务器时
* 做所谓的 checkpoint 时

## 六、问题
### 1、为什么redolog比正常数据库写磁盘快
* 因为普通数据写磁盘是随机的，会慢，但是log日志是追加模式的，是一种顺序IO。

* 还有就是buffer是数据页为单位的，大小为16K,一个数据页上一个小小的修改，都要把整个数据页写入。而redolog只需要写入真正需要的部分就可以了。无效的IO大大减少了。