## goroutine
在java/c++中我们要实现并发编程的时候，通常需要自己维护一个线程池，并且需要自己去包装一个又一个的任务，同时需要自己去调度线程执行任务并维护上下文切换，这一切通常会耗费程序员大量的心智。

goroutine的概念类似于线程，但 **goroutine是由Go的运行时（runtime）调度和管理的**。Go程序会智能地将 goroutine 中的任务合理地分配给每个CPU。Go语言之所以被称为现代化的编程语言，就是因为**它在语言层面已经内置了调度和上下文切换的机制**。

goroutine的精髓就是**控制流的主动让出和恢复。**

在Go语言编程中你不需要去自己写进程、线程、协程，你的技能包里只有一个技能–goroutine，当你需要让某个任务并发执行的时候，你只需要把这个任务包装成一个函数，开启一个goroutine去执行这个函数就可以了，就是这么简单粗暴。

goroutine 只是由官方实现的超级"线程池"。

每个实力4~5KB的栈内存占用和由于实现机制而大幅减少的创建和销毁开销是go高并发的根本原因。

## 一、基础知识
### 1、goroutine使用
```go
func main() {
    go hello() // 启动另外一个goroutine去执行hello函数
    fmt.Println("main goroutine done!")
    //time.Sleep(time.Second)
}
```
结果只打印了main goroutine done!，并没有打印Hello Goroutine!。

**在程序启动时，Go程序就会为main()函数创建一个默认的goroutine。**

当main()函数返回的时候**该**goroutine就结束了，所有在main()函数中启动的goroutine会一同结束。

所以我们要想办法让main函数等一等hello函数，最简单粗暴的方式就是time.Sleep了。


### goroutine的定义

* 任何函数只需加上go就能送给调度器运行
* 不需要再定义时区分是否是异步函数
* 调度器在合适的点进行切换
* 使用-race来检查数据冲突 todo
* go的协程调度由程序处理，开发者不用关心

### goroutine可能的切换点（参考：不能保证切换，也不能保证在其他地方不切换）
* I/O, select
* channel
* 等待锁
* 函数调用(有时)
* runtime.Gosched()

##  其他
* 互斥锁，读写锁
* map 不是并发安全的，即不支持并发写。Go语言的sync包中提供了一个开箱即用的并发安全版map–sync.Map。开箱即用表示不用像内置的map一样使用make函数初始化就能直接使用。同时sync.Map内置了诸如Store、Load、LoadOrStore、Delete、Range等操作方法。 todo

 