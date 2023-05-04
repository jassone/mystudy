## channel底层原理

 ## 一、底层结构

```go
   type hchan struct {
       qcount   uint           // total data in the queue 当前队列里还剩余元素个数
       dataqsiz uint           // size of the circular queue 环形队列长度，即缓冲区的大小，即make(chan T,N) 中的N
       buf      unsafe.Pointer // points to an array of dataqsiz elements 环形队列指针，循环队列是大小固定的用来存放chan接收的数据的队列；
       elemsize uint16 //每个元素的大小，用于在 buf 中定位元素的位置
       elemtype *_type // element type 元素类型，用于数据传递过程中的赋值
       closed   uint32 //标识当前通道是否处于关闭状态，创建通道后，该字段设置0，即打开通道；通道调用close将其设置为1，通道关闭
       sendx    uint   // send index 环形缓冲区的状态字段，待发送的数据在循环队列buffer中的位置索引；
       recvx    uint   // receive index 环形缓冲区的状态字段，待接收的数据在循环队列buffer中的位置索引
       recvq    waitq  // list of recv waiters 等待接受的goroutine队列
       sendq    waitq  // list of send waiters 等待发送的goroutine队列
    
       // lock protects all fields in hchan, as well as several
       // fields in sudogs blocked on this channel.
       //
       // Do not change another G's status while holding this lock
       // (in particular, do not ready a G), as this can deadlock
       // with stack shrinking.
       lock mutex //互斥锁，为每个读写操作锁定通道，因为发送和接受必须是互斥操作
  }
  
  // sudog 代表goroutine
   type waitq struct {
        first *sudog
        last  *sudog
  }
```

![oc2lamvaji.png](https://pic1.imgdb.cn/item/6443c2e70d2dde577741e99f.png)

## 二、操作

### 1、带缓冲区的结构

比如缓冲区是6，已经加入了两个元素了，**就是一个环形队列**。

![C9E52084B22B.png](https://pic1.imgdb.cn/item/641575a5a682492fcc4da0bf.png)

```go
   ch:=make(chan int, 3)
```

创建通道后的缓冲通道结构

```go
   hchan struct {
    qcount uint : 0 
    dataqsiz uint : 3 
    buf unsafe.Pointer : 0xc00007e0e0 
    elemsize uint16 : 8 
    closed uint32 : 0 
    elemtype *runtime._type : &{
        size:8 
        ptrdata:0 
        hash:4149441018 
        tflag:7 
        align:8 
        fieldalign:8 
        kind:130 
        alg:0x55cdf0 
        gcdata:0x4d61b4 
        str:1055 
        ptrToThis:45152
        }
    sendx uint : 0 
    recvx uint : 0 
    recvq runtime.waitq : 
        {first:<nil> last:<nil>}
    sendq runtime.waitq : 
        {first:<nil> last:<nil>}
    lock runtime.mutex : 
        {key:0}
}
```

源码在$GOPATH/src/runtime/chan.go下：

```go
   func makechan(t *chantype, size int) *hchan {
           elem := t.elem
           ...
    }
```

创建一个带有buffer的channel，底层的数据结构模型如图：

![1430c1feeed649c3.png](https://pic1.imgdb.cn/item/64156803a682492fcc33a5a1.webp)

### 2、向channel中写入数据

```go
ch <- 3
```

 发送操作步骤：

- 锁定整个通道结构（比如对buf加锁，**将数据copy到buf中**，然后sendx++，然后释放对buf的锁，因为缓存链表中以上每一步的操作，都是需要加锁操作的）。
- 先判断等待接收队列 recvq 中是否为空，如果 recvq 不为空，说明之前在进行读操作时，缓冲区中没有数据或者没有缓冲区，所以造成了阻塞并挂在到等待接收队列中，此时直接从 recvq 取出 G，并把数据写入，最后把该 G 唤醒，结束写过程
- 如果等待接收队列为空，并且缓冲区中有空余位置，那就直接将数据写入缓冲区，结束发送过程；
- 如果等待接收队列为空，并且缓冲区中没有空余位置，将待发送数据写入G，将当前G加入sendq，进入睡眠，等待被读goroutine唤醒；
- 写入完成释放锁

执行流程图如下：

 [![fd629ad6b.webp](https://pic1.imgdb.cn/item/6443bdc30d2dde57773c0a0a.webp)

### 3、从channel中读取数据

```go
val := <- a
```

底层hchan数据流转如下图:

 ![bcc9a526.png](https://pic1.imgdb.cn/item/64157a59a682492fcc57e5f2.webp)

读数据操作如下:

- 先获取channel全局锁。（比如对buf加锁，然后将buf中的**数据copy到task变量对应的内存里**，然后recvx++,并释放锁）
- 尝试sendq等待队列中获取等待的goroutine
- 如果有等待的goroutine，没有缓冲区，取出goroutine并读取数据，然后唤醒这个goroutine，结束读取释放锁
- 如果有等待goroutine，且有缓冲区(缓冲区满了)，从缓冲区队列首取数据，再从sendq取出一个goroutine，将goroutine中的数据存放到buf队列尾，结束读取释放锁。
- 如果没有等待的goroutine，且缓冲区有数据，直接读取缓冲区数据，结束释放锁。
- 如果没有等待的goroutine，且没有缓冲区或者缓冲区为空，将当前goroutine加入到sendq队列，进入睡眠，等待被写入goroutine唤醒，结束读取释放锁。

### 3、等待队列
- 从channel 读数据时如果channel缓冲区为空，当前goroutine 会被阻塞；
- 写数据时，如果channel缓冲区为满，当前goroutine 会被阻塞，
- 还有在进行读或者写操作时没有缓冲区也会使当前goroutine阻塞

被阻塞的goroutine会被挂在channel的等待队列中，唤醒条件：

- 因为读阻塞的 goroutine ：向 channel 写入数据的 goroutine 唤醒
- 因为写阻塞的 goroutine ：从 channel 读数据的 goroutine 唤醒

一般情况下 recvq 和 sendq 至少有一个为空；只有一个例外，就是同一个 goroutine 使用 select 语句向 channel 一边写数据一边读数据

## 三、recvq和sendq

recvq和sendq基本上是链表，基本如下：

    recvq    waitq  // list of recv waiters 等待接受的goroutine队列
    sendq    waitq  // list of send waiters 等待发送的goroutine队列
    
    type waitq struct {
    	first *sudog
    	last  *sudog
    }

![c729f5ca.png](https://pic1.imgdb.cn/item/64158dcfa682492fcc807cbf.webp) 

sendq 和 recvq 存储了当前 Channel 由于缓冲区空间不足而阻塞的 Goroutine 列表，这些等待队列使用双向链表 waitq 表示，链表中所有的元素都是 sudog 结构。

sudog代表着等待队列中的一个goroutine，G与同步对象（指chan）关系是多对多的。一个 G 可以出现在许多等待队列上，因此一个 G 可能有多个sudog。并且多个 G 可能正在等待同一个同步对象，因此一个对象可能有许多 sudog。sudog 是从特殊池中分配出来的。使用 acquireSudog 和 releaseSudog 分配和释放它们。

![jonx1ujg5l.png](https://pic1.imgdb.cn/item/6443c45d0d2dde5777436947.png)

这里其实主要需要明确两点：

- channel中的数据遵循队列**先进先出原则**。
- 每一个goroutine抢到处理器的时间点不一致，gorouine的执行本身不能保证顺序。

即代码中先写的gorouine并不能保证先从channel中获取数据，或者发送数据。但是先执行的gorouine与后执行的goroutine在channel中获取的数据肯定是有序的。

## 四、细节

### 1、Channel为什么是线程安全的

在对buf中的数据进行入队和出队操作时，为当前chnnel使用了互斥锁，防止多个线程并发修改数据。

## 五、相关wiki

- 深入分析Go1.18 Channel底层原理 https://cloud.tencent.com/developer/article/2126558
