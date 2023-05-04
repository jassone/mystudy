## lazy free

lazy free 对应了 4 种场景，默认都是关闭的：

- lazyfree-lazy-eviction  no

  表示当 Redis 运行内存超过 maxmeory 时，是否开启 lazy free 机制删除；

- lazyfree-lazy-expire  no

  表示设置了过期时间的键值，当过期之后是否开启 lazy free 机制删除；

- lazyfree-lazy-server-del  no

  有些指令在处理已存在的键时，会带有一个隐式的 del 键的操作，比如 rename 命令，当目标键已存在，Redis 会先删除目标键，如果这些目标键是一个 big key，就会造成阻塞删除的问题，此配置表示在这种场景中是否开启 lazy free 机制删除；

- slave-lazy-flush   no

  针对 slave(从节点) 进行全量数据同步，slave 在加载 master 的 RDB 文件前，会运行 flushall 来清理自己的数据，它表示此时是否开启 lazy free 机制删除。



todo

https://www.jb51.net/article/163919.htm