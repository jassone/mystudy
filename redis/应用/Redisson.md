## Redisson

Redisson是架设在Redis基础上的一个Java常驻内存数据网格（In-Memory Data Grid）。

java的redis客户端
* Jedis
* Lettuce 

## 一、常用功能
### 1、提供**分布式锁**和同步器
可重入锁（Reentrant Lock）、公平锁（Fair Lock）、联锁（MultiLock）、 红锁（RedLock）、读写锁（ReadWriteLock）、信号量（Semaphore）、可过期性信号量（PermitExpirableSemaphore）和闭锁（CountDownLatch）

分布式锁
![20220420154830.jpg](https://pic.imgdb.cn/item/625fbb32239250f7c5b9f6f9.jpg)
支持可重入锁
![20220420154905.jpg](https://pic.imgdb.cn/item/625fbb5c239250f7c5ba4a1b.jpg)

### 2、提供分布式集合(比如**延迟列队**)
映射（Map）、多值映射（Multimap）、集（Set）、列表（List）、有序集（SortedSet）、计分排序集（ScoredSortedSet）、字典排序集（LexSortedSet）、列队（Queue）、双端队列（Deque）、阻塞队列（Blocking Queue）、有界阻塞列队（Bounded Blocking Queue）、 阻塞双端列队（Blocking Deque）、阻塞公平列队（Blocking Fair Queue）、**延迟列队**（Delayed Queue）、优先队列（Priority Queue）和优先双端队列（Priority Deque）

### 2、使用redis简单实现分布式锁

```sh
SET key value [EX seconds] [PX milliseconds] [NX|XX] 
```
这个命令， 就是 set k v nx px ttl 这种命令。

**但是不能重复加锁(重入锁)。**

### 3、相关wiki
* Redisson Redis 客户端 https://www.oschina.net/p/redisson?hmsr=aladdin1e1