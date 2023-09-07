## singleflight设计

todo



## go中的设计

https://www.jianshu.com/p/ec59beec138b

项目： golang.org/x/sync/singleflight，底层原理：锁+内存共享

只适用于单实例或者单进程的场景。

## 实现分布式singleflight

借助redis

- setnx来实现分布式锁。
- 通过Redis共享回源结果。

注意：可以适当允许多个实例抢锁回源，提高成功率(可能单个会失败)。

