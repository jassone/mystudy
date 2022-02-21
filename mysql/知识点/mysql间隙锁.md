## mysql间隙锁

### InnoDB使用间隙锁的目的有2个：
* **为了防止幻读(上面也说了，Repeatable read隔离级别下再通过GAP锁即可避免了幻读)**

* 满足恢复和复制的需要：
    MySQL 通过 BINLOG 录入执行成功的 INSERT、UPDATE、DELETE 等更新数据的 SQL 语句，并由此实现 MySQL 数据库的恢复和主从复制。MySQL 的恢复机制（复制其实就是在 Slave Mysql 不断做基于 BINLOG 的恢复）有以下特点：
    
    一是 MySQL 的恢复是 SQL 语句级的，也就是重新执行 BINLOG 中的 SQL 语句。
    
    二是 MySQL 的 Binlog 是按照事务提交的先后顺序记录的， 恢复也是按这个顺序进行的。
    
    由此可见，MySQL 的恢复机制要求：**在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，也就是不允许出现幻读。**
    
### todo
https://www.jianshu.com/p/32904ee07e56
 
