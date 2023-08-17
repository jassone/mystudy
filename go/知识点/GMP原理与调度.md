## GMP原理与调度
非常经典

https://www.yuque.com/aceld/golang/srxd6d

## 一、简单总结

G代表Goroutine协程、M代表内核线程、P代表上下文处理器，调度器
P的数量由GOMAXPROCS决定，程序运行就创建好了这么多个P。
当一个新的G被创建的时候，会优先将G放入P的本地队列，满了才会放入全局队列。
**多线程复用（work stealing、hand off机制）：**

- P开始执行G的时候，先从本地队列中获取G，如果没有，就去全局队列拿一些，如果没有，则去其他的P中偷取一些G来执行。
- M和P在执行的时候是1:1绑定的，真正的执行操作是由M内核线程来完成的，操作系统只能感知到M，然后将M放入CPU执行。当M因为G的一些系统调用阻塞的是时候，会释放P。然后runtime会创建新的线程、或者复用空闲的M来服务P。
- 如果一个M执行完了G任务，去寻找有没有空闲的P，如果没有就会进入休眠，等待被唤醒。

**抢占式调度（公平性）**

- MP执行G是通过轮询的方式，每个G只能执行10ms，防止其他G饥饿。
- 抢占式调度只会在垃圾回收扫描任务时触发？？？ 高版本是基于信号的抢占调度？？？ Todo

![](https://pic.imgdb.cn/item/64b79af11ddac507cc77fad3.png)



**利用多核并行**

- 所以，同一时间，只能有GOMAXPROCS的Goroutine在同时运行，因为只有这么多个P。

## 二、其他

- 认识Golang中的sysmon监控线程 https://blog.haohtml.com/archives/22745  todo