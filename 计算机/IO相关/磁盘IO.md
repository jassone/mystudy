## 磁盘IO
## 一、磁盘IO分类
* 缓冲IO（buffered IO）
* 无缓冲IO（unbuffered IO）
* 直接IO（Direct IO）

|类型|对应API接口|
|---|---|
|缓冲IO|c语言的库函数(支持linux和win)：fopen,fclose,fseek,fread,fwrite,fprintf,fscanf...|
|无缓冲IO|linux系统特有API:open,close,lseek,fsync,read,write,pread,pwrite|

* 直接IO,就是直接读写磁盘，不经过操作系统的Pagecache。
* 直接IO和无缓冲IO所用的API是一样的，只是在open文件时加了O_DIRECT参数。

### 1、有缓存IO和无缓存IO的对比
![20220226214022.jpg](https://pic.imgdb.cn/item/621a2df22ab3f51d91c25f51.jpg)
无缓存IO只是用户空间无缓存。其他和有缓存IO一样。

## 二、IO过程
整个IO过程中间都经历了什么
![20220226203047.jpg](https://pic.imgdb.cn/item/621a1d812ab3f51d91984810.jpg)
从最顶层到最底层，每一层都可能有缓存。

* 应用程序内存：通常写代码用malloc/free(c)、new/delete(C++)等分配出来的内存。

* 用户缓冲区：C语言的的FILE结构体里面的buffer

    ```c
typedef struct  {
    short           level;          /* fill/empty level of buffer */
    unsigned        flags;          /* File status flags    */
    char            fd;             /* File descriptor      */
    unsigned char   hold;           /* Ungetc char if no buffer */
    short           bsize;          /* Buffer size          */
    unsigned char   *buffer;        /* Data transfer buffer */
    unsigned char   *curp;          /* Current active pointer */
    unsigned        istemp;         /* Temporary file indicator */
    short           token;          /* Used for validity checking */
}  FILE;                           /* This is the FILE object */
```

* 内核缓冲区：linux操作系统的**Pagecache**。为了加快磁盘IO,linux系统会把磁盘上的数据以Page为单位缓存在操作系统的内存里，这里的Page是linux系统定义的一个逻辑概念，一个Page一般是4K。

### 1、快设备
以‘快’为最小读写单位，快大小为512字节(传统机型硬盘，一个扇区大小就是512字节)
内存是‘字节设备’，最小读取单位是1字节。

### 2、物理扇区/逻辑扇区
物理扇区：就是传统机械硬盘的扇区，512字节。

逻辑扇区：现代的机械硬盘扇区已达4K，同时SSD硬盘也没有扇区概念，但在操作系统里面，一个扇区512字节的概念已经根深蒂固。为了兼容，有了‘逻辑扇区’概念，逻辑扇区统一传统、现代机械硬盘、SSD硬盘，还是512字节。

### 3、通用快层
把各种异构的磁盘设备，抽象成通用的一个个‘快’，供上层文件系统调用。在这层会对IO请求进行排队、合并，尽可能提高IO性能。

### 4、fflush/fflush
* fflush:缓冲IO中的一个API,它只是把数据从用户缓冲区刷到内核缓冲区而已。

* fsync:把数据从内核缓冲区刷到磁盘。

##### a) 注意
* 无论是缓冲IO还是无缓冲IO，如果再写数据之后不调用fsync，此时系统断电后重启，最新的部分数据会丢失。
* 如果没出IO都调用fsync,性能会在数量级上的下降。

##### b) 性能和可靠性权衡
* mysql的双1参数,牺牲一定性能，保证数据可靠性
    ```ini
innodb_flush_log_at_trx_commit=1 //每次事务提交，innodb redolog调用fsync刷盘
sysc_binlog=1 //每次事务提交，binlog调用fsync刷盘
```

* kafka,为了提高性能，异步刷盘，因为有3个副本，可以忍受1个副本丢数据
    
    ```ini
log.flush.interval.messages //没多少条刷盘一次
log.flush.interval.ms //每隔多长时间，刷盘一次
log.flush.scheduler.interval.ms //周期性刷盘，默认3s
```

## 三、内存映射文件
### 1、对应API
* linux API:

    ```c
void* mmap(void* start,size_t length,int prot,int flags,**int fd**,off_t offset);
```
* java API：MappedByteBuffer类可以实现同样的目的。

![20220226231847.jpg](https://pic.imgdb.cn/item/621a44df2ab3f51d910336e9.jpg)
直接拿应用程序的逻辑内存地址映射到linux操作系统的内核缓冲区，应用程序虽然读写的是自己的内存，但这个内存只是一个‘逻辑地址’，实际读写的是内核缓冲区。

## 四、零拷贝
### 1、未使用零拷贝
如果有这样一个场景：我们需要读取磁盘数据，再通过网络发送给其他机器。则流程如下

![20220226232205.jpg](https://pic.imgdb.cn/item/621a45a42ab3f51d91054fb1.jpg)
伪代码：
* fd1=打开的文件描述符
* fd2=打开的socket描述符
* buffer=应用程序内存
* read(fd1,buffer...)//先把数据从文件中读出来
* write(fd2,buffer...)//再通过网络发出去

整个过程会有4次数据拷贝，读进来2次，写回去2次。
```
磁盘->内核缓冲区->应用程序内存->socket缓存区->网络
```

### 2、使用零拷贝
![20220226232650.jpg](https://pic.imgdb.cn/item/621a46c22ab3f51d91085e3f.jpg)
虽然叫零拷贝，实际是2次数据拷贝，1次是从磁盘到内核缓存区，1次是从内核缓存区到网络。之所以叫零拷贝，是从内存的角度来看的，数据再内存中没有发生过数据拷贝，只在内存和IO之间传输。

##### a) 对应API
* linux API：

    ```c
sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```
out_fd是socket描述符，in_fd是文件描述符。

* java API： 
    ```java
FileChannel.transferTo(long position, long count, WritableByteChannel target)
```

##### b）典型应用-kafka consumer
kafka consumer消费消息的时候，Broker通过零拷贝技术把磁盘上的消息读出来，然后通过网络发送给Consumer。

## 五、5种磁盘IO的主要应用场景
![20220226234014.jpg](https://pic.imgdb.cn/item/621a49e52ab3f51d9111006a.jpg)

## 六、其他
* 磁盘读取依靠的是机械运动，分为寻道时间、旋转延迟、传输时间三个部分，这三个部分耗时相加就是一次磁盘IO的时间，大概`9ms`左右。

