## channel
单纯地函数并发执行是没有意义的。函数与函数间需要交换数据才能体现并发执行函数的意义。

虽然可以使用共享内存进行数据交换，但是共享内存在不同的goroutine中容易发生竞态问题。为了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必造成性能问题。

Go语言的并发模型是CSP（Communicating Sequential Processes），**提倡通过通信共享内存而不是通过共享内存而实现通信。**

**如果说goroutine是Go程序并发的执行体，channel就是它们之间的连接。channel是可以让一个goroutine发送特定值到另一个goroutine的通信机制。**

Go 语言中的通道（channel）是一种特殊的类型。通道像一个传送带或者队列，总是**遵循先入先出（First In First Out）的规则，保证收发数据的顺序。**每一个通道都是一个具体类型的导管，也就是声明channel的时候需要为其指定元素类型。

## 一、基础知识

### 1、channel声明

channel是一种类型，一种引用类型，通道类型的空值是nil。声明通道类型的格式如下：
```go
// 声明
var ch1 chan 元素类型
//声明的通道后需要使用make函数初始化之后才能使用。
ch1 = make(chan 元素类型)
```

### 2、channel创建-初始化

channel创建的格式如下：make(chan 元素类型, [缓冲大小])
```go 
var ch chan int
ch = make(chan int)
fmt.Println(ch)

//或者声明创建一起
ch2 := make(chan int)
```
### 3、channel操作
通道有发送（send）、接收(receive）和关闭（close）三种操作。

* ch <-  // **写 channel 是必须带上值**

##### a) close
通道是可以被垃圾回收机制回收的，关闭通道不是必须的。

但是**管道不往里存值或者取值的时候一定记得关闭管道**。

*   对一个关闭的通道进行接收一直获取值，当为空时返回零值。
*   i, ok := <-ch1 // 通道关闭后再取值ok=false，可以用于判断通道是否已关闭。

**注意：对于已经明确知道不需要的channel，需要关闭，否则当前goroutine执行完后，只剩下main函数中接受channel，会报死锁。**

### 4、无缓冲的通道
```go
ch := make(chan int)
ch <- 10
```
上面这段代码能够通过编译，但是执行的时候会出现以下错误：
```sh
fatal error: all goroutines are asleep - deadlock!
# 死锁原因：当操作通道读写时，前面没有协程在跑(只要前面有协程就行，不用管协程中是否会处理当前的通道).
```

因为我们使用ch := make(chan int)创建的是无缓冲的通道，**无缓冲的通道只有在有人接收值的时候才能发送值。**

上面的代码会**阻塞在ch <- 10这一行代码形成死锁**，那如何解决这个问题呢？

一种方法是启用一个goroutine去接收值，例如：
```go
func recv(c chan int) {
    ret := <-c
    fmt.Println("接收成功", ret)
}
func main() {
    ch := make(chan int)
    go recv(ch) // 启用goroutine从通道接收值
    ch <- 10
    fmt.Println("发送成功")
}
```

使用**无缓冲通道进行通信将导致发送和接收的goroutine同步化**。因此，**无缓冲通道也被称为同步通道。**

### 5、有缓冲的通道

```go
func main() {
    ch := make(chan int, 2) // 创建一个容量为2的有缓冲区通道
    ch <- 10
    fmt.Println("发送成功")
}
```
可以使用内置的len函数获取通道内元素的数量，使用cap函数获取通道的容量。

### 6、优雅的从通道循环取值
```go
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    go func() {
        for i := 0; i < 100; i++ {
            ch1 <- i
        }
        close(ch1)
    }()
    // 开启goroutine从ch1中接收值，并将该值的平方发送到ch2中
    go func() {
        for {
            i, ok := <-ch1 // 通道关闭后再取值ok=false
            if !ok {
                break
            }
            ch2 <- i * i
        }
        close(ch2)
    }()
    // 在主goroutine中从ch2中接收值打印
    for i := range ch2 { // 通道关闭后会读出内容，再退出for range循环
        fmt.Println(i)
    }
}
```
range用的比较多。**而且注意这里是循环的通道列表，不是单个通道内容**。

### 7、单向通道

chan<- 表示数据进入通道    <-chan 表示数据从通道出来。

```go
func demo()  <-chan int {
   out := make(chan int)
   go func() {
      out <- 1
   }()
   
   return out // 这里的out还是双向的，只是赋值给返回值后，就是单向的了
}
```

**有方向的 channel 不可以被关闭。**

```go
func Stop(stop <-chan bool) {
	close(stop) // panic
}
```

### 8、异常情况总结

 ![channel01.png](https://pic.imgdb.cn/item/622d3d555baa1a80ab00ec67.png)

* 关闭已经关闭的channel会引发panic。
* 对一个关闭的通道再发送值会导致panic。

口诀：nil读写阻塞，写关闭/关关闭异常，读关闭空零

### 10、channel的妙用

- **传递方面：消息传递，任务发送，事件广播**
- **控制方面：资源争抢，并发控制，流程开关**

##### channel的应用场景

1. 协程之间的通信. 通过channel传递数据,实现通信
2. 协程之间的同步. 通过控制channel读写数据的时序,实现不同协程之间的同步
3. 实现工作池. 通过channel实现对工作池任务的分配和处理
4. 实现信号量. 设置1大小的缓冲区,模拟加锁和解锁
5. 实现超时机制. channel结合select实现超时任务.

### 11、如何利用channel阻塞

- 实现资源争抢，谁抢到就是谁的
- 实现多协程锁，变相的‘锁’
- 实现消息定额消费，生产者和消费者的关系
