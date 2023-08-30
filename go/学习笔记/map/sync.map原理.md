## sync.map原理



todo



优缺点
优点：Go官方所出；通过读写分离，降低锁时间来提高效率；
缺点：不适用于大量写的场景，这样会导致 read map 读不到数据而进一步加锁读取，同时dirty map也会一直晋升为read map，整体性能较差，甚至没有单纯的 map+metux 高。
适用场景：读多写少的场景。



https://blog.csdn.net/qq_37102984/article/details/128154924