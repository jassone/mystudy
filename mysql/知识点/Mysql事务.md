## Mysql事务
## 一、四大属性
事务具有四个特征：原子性（ Atomicity ）、隔离性（ Isolation ）和持续性（ Durability ）、一致性（ Consistency ）。这四个特性简称为 ACID 特性。

## 二、原子性
事务是数据库的逻辑工作单位，事务中包含的各操作要么都做，要么都不做，也就是说事务是一个不可分割的整体，就像化学元素原子一样，最小的单位
> 通过undo log保证。

### 1 undo log是什么
Undo log 主要用于**记录数据被修改之前的日志，在表信息修改之前先会把数据拷贝到undolog里。**

当事务进行回滚时可以通过undo log 里的日志进行数据还原(记录了相反的操作)。

### 2 Undo log 的用途
* 保证事务进行rollback时的原子性和一致性，当事务进行回滚的时候可以用undo log的数据进行恢复。

* 用于MVCC快照读的数据，在MVCC多版本控制中，通过读取undo log的历史版本数据可以实现不同事务版本号都拥有自己独立的快照数据版本。

### 3 undo log主要分为两种：

* insert undo log
代表事务在insert新记录时产生的undo log , 只在事务回滚时需要，并且在事务提交后可以被**立即丢弃**。

* update undo log（主要）
  事务在进行update或delete时产生的undo log; 不仅在事务回滚时需要，在快照读时也需要。

  所以不能随便删除，只有在快速读或事务回滚不涉及该日志时，对应的日志才会被purge线程统一清除。

## 三、隔离性
一个事务的执行不能被其它事务干扰。同一时间，只允许一个事务请求同一数据，不同事务之间彼此没有任何干扰，其它的状态转换不会影响到本次状态转换。
> 通过MVCC保证。

## 四、 持续性
也称永久性，指一个事务一旦提交，它对数据库中的数据的改变就应该是永久性的。接下来的其它操作或故障不应该对其执行结果有任何影响。 既事务完成后，事务对数据库的所有更新被保存到数据库，不能回滚。
> 通过redo log保证。

redo log记录了修改完以后的数据。

## 五、一致性
**上面三个属性一起保证一致性**。既事务开始前和结束后，数据库完整性约束没有被破坏；事务执行的结果必须是使数据库从一个一致性状态变到另一个一致性状态。因此当数据库只包含成功事务提交的结果时，就说数据库处于一致性状态。如果数据库系统 运行中发生故障，有些事务尚未完成就被迫中断，这些未完成事务对数据库所做的修改有一部分已写入物理数据库，这时数据库就处于一种不正确的状态，或者说是 不一致的状态。

### 1、事务回滚正确的理解

人们对事务的解释如下：事务由作为一个单独单元的一个或多个SQL语句组成，如果其中一个语句不能完成，整个单元就会回滚（撤销），所有影响到的数据将返回到事务开始以前的状态。因而，只有事务中的所有语句都成功地执行才能说这个事务被成功地执行。

如果事务中两条语句中的一条执行失败了，但如果以后在该连接中又执行了一条commit 或begin或start transaction（新开一个事务会将该链接中的其他未提交的事务提交，相当于commit！），即成功的那条语句被执行成功了。

所以事务回滚正确的理解应该是，如果事务中所有sql语句执行正确则需要自己手动提交commit；否则有任何一条执行错误，需要自己提交一条rollback，这时会回滚所有操作，而不是commit会给你自动判断和回滚。

## 六、事务状态
### 6-1、活动的（active）
事务对应的数据库操作正在执行过程中时，我们就说该事务处在 活动的 状态。
### 6-2、部分提交的（partially committed）
当事务中的最后一个操作执行完成，但由于操作都在内存中执行，所造成的影响并没有刷新到磁盘时，我们
就说该事务处在 部分提交的 状态。
### 6-3、失败的（failed）
当事务处在 活动的 或者 部分提交的 状态时，可能遇到了某些错误（数据库自身的错误、操作系统错误或者
直接断电等）而无法继续执行，或者人为的停止当前事务的执行，我们就说该事务处在 失败的 状态。
### 6-4、中止的（aborted）
如果事务执行了半截而变为 失败的 状态，就是要撤销失败事务对当前数据库造成的影响。书面一点的话，我们把这个撤销的过程称之为 回滚 。当 回滚 操作执行完毕时，也就是数据库恢复到了执行事务之前的状态，我们就说该事务处在了 中止的 状态。
### 6-5、提交的（committed）
当一个处在 部分提交的 状态的事务将修改过的数据都同步到磁盘上之后，我们就可以说该事务处在了 提交 的状态。

### 说明
**只有当事务处于提交的或者中止的状态时，一个事务的生命周期才算是结束了**。

## 七、MySQL中事务的语法
### 7-1、开启事务

##### BEGIN [WORK];
BEGIN 语句代表开启一个事务，后边的单词WORK 可有可无。开启事务后，就可以继续写若干条语句，这些语句都属于刚刚开启的这个事务。

```sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)
mysql> 加入事务的语句...
```
##### START TRANSACTION
同BEGIN， 但是他语句后边跟随几个 修饰符：
READ ONLY，READ WRITE，WITH CONSISTENT SNAPSHOT(**会在执行该语句后立即生成一个ReadView**，而不是在执行第一条select语句时才生成)。

### 7-2、提交事务
```sql
COMMIT [WORK]
```

### 7-3、手动中止事务
```sql
ROLLBACK [WORK]
```

## 八、事务其他属性
### 自动提交
mysql有个系统变量autocommit，默认是off.

如果要关闭，有两个办法
* 显式的的使用 START TRANSACTION 或者 BEGIN 语句开启一个事务。这样在本次事务提交或者回滚前会暂时关闭掉自动提交的功能。
* 把系统变量 autocommit 的值设置为 OFF ，就像这样：SET autocommit = OFF;

###  隐式提交
当我们使用 START TRANSACTION 或者 BEGIN 语句开启了一个事务，或者把系统变量 autocommit 的值设置为 OFF时，事务就不会进行 自动提交 ，但是如果我们输入了某些语句之后就会 悄悄的 提交掉，就像我们输入了COMMIT 语句了一样，这种因为某些特殊的语句而导致事务提交的情况称为 隐式提交 。

这些会导致事务隐式提交的语句包括：
* 定义或修改数据库对象的数据定义语言（Data definition language，缩写为： DDL ）。

    所谓的数据库对象，指的就是 数据库 、 表 、 视图 、 存储过程 等等这些东西。当我们使用 CREATE 、ALTER 、 DROP 等语句去修改这些所谓的数据库对象时，就会隐式的提交前边语句所属于的事务，

* 隐式使用或修改 mysql 数据库中的表
    当我们使用 ALTER USER 、 CREATE USER 、 DROP USER 、 GRANT 、 RENAME USER 、 REVOKE 、 SET PASSWORD 等语句时也会隐式的提交前边语句所属于的事务。

* 事务控制或关于锁定的语句
    当我们在一个事务还没提交或者回滚时就又使用 START TRANSACTION 或者 BEGIN 语句开启了另一个事务时，会隐式的提交上一个事务。
    
    或者当前的 autocommit 系统变量的值为 OFF ，我们手动把它调为 ON 时，也会隐式的提交前边语句所属的事务。
    或者使用 LOCK TABLES 、 UNLOCK TABLES 等关于锁定的语句也会隐式的提交前边语句所属的事务。
    
* 加载数据的语句
    比如我们使用 LOAD DATA 语句来批量往数据库中导入数据时，也会隐式的提交前边语句所属的事务。
    
* 关于 MySQL 复制的一些语句
    使用 START SLAVE 、 STOP SLAVE 、 RESET SLAVE 、 CHANGE MASTER TO 等语句时也会隐式的提交前边语句
    所属的事务。

* 其它的一些语句
    使用 ANALYZE TABLE 、 CACHE INDEX 、 CHECK TABLE 、 FLUSH 、 LOAD INDEX INTO CACHE 、 OPTIMIZE
    TABLE 、 REPAIR TABLE 、 RESET 等语句也会隐式的提交前边语句所属的事务。

### 其他
* 主键的自动递增是不会回滚的

## 九、保存点
我们在调用 ROLLBACK 语句时可以指定会滚到哪个点，而不是回到最初的原点。定义保存点的语法如下：

```sql
SAVEPOINT 保存点名称;
```

当我们想回滚到某个保存点时，可以使用下边这个语句（下边语句中的单词 WORK 和 SAVEPOINT 是可有可无的）：

```
ROLLBACK [WORK] TO [SAVEPOINT] 保存点名称;
```