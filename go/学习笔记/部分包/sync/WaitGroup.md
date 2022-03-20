## WaitGroup
**WaitGroup用于等待一组线程的结束。**

父线程调用Add方法来设定应等待的线程的数量。每个被等待的线程在结束时应调用Done方法。同时，主线程里可以调用Wait方法阻塞至所有线程结束。

### 1、主要函数
```go
func (wg *WaitGroup) Add(delta int)
```

Add方法向内部计数加上delta，delta可以是负数；如果内部计数器变为0，Wait方法阻塞等待的所有协程都会释放，如果计数器小于0，则调用panic。注意Add加上正数的调用应在Wait之前，否则Wait可能只会等待很少的协程。一般来说本方法应在创建新的协程或者其他应等待的事件之前调用。

```go
func (wg *WaitGroup) Done()
```
Done方法减少WaitGroup计数器的值，应在协程的最后执行。

```go
func (wg *WaitGroup) Wait() 
```
Wait方法阻塞代码的运行直到WaitGroup计数器减为0。
 
### 2、小结
以上程序通过WaitGroup提供的三个同步接口，实现了等待一个协程组完成的同步操作。在实现时要注意：
* Add的参数N必须和创建的goroutine的数量相等，否则会报出死锁的错误信息
* 函数中要传递WaitGroup需要用指针传递，在该结构的实现源码中已经有讲解：
A WaitGroup must not be copied after first use.

就是说，该结构定义后就不能被复制，所以这里要使用指针。

sync.WaitGroup是一个结构体，传递的时候要传递指针。

### 3、不足
WaitGroup在需要等待多个任务结束再返回的业务来说还是很有用的，但现实中用的更多的可能是，先等待一个协程组，若所有协程组都正确完成，则一直等到所有协程组结束；若其中有一个协程发生错误，则需要告诉协程组的其他协程，全部停止运行（本次任务失败）以免浪费系统资源。

该场景WaitGroup是无法实现的，那么该场景该如何实现呢，就需要用到通知机制，其实也可以用channel来实现。
