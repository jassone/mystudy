## bufio

### bufio目的是通过缓冲来提高效率。

简单的说就是，**把文件读取进缓冲（内存）之后再读取的时候就可以避免文件系统的io 从而提高速度**。同理，在进行写操作时，先把文件写入缓冲（内存），然后由缓冲写入文件系统。看完以上解释有人可能会表示困惑了，直接把 内容->文件 和 内容->缓冲->文件相比， 缓冲区好像没有起到作用嘛。其实缓冲区的设计是为了存储多次的写入，最后一口气把缓冲区内容写入文件。

![28efa.webp](https://pic1.imgdb.cn/item/63316c3816f2c2beb1b48f4c.png)

## 细节

### 1、Flush()

buff默认为4096字节，在代码中，最后需要用Flush()把缓冲区剩下的数据刷到文件中。

## 相关wiki

- go bufio缓冲io详解 https://blog.51cto.com/ghostwritten/5345287
- go bufio https://www.jianshu.com/p/04aebd42b762
- [Golang学习 - bufio 包] https://www.cnblogs.com/golove/p/3282667.html