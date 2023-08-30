## waitgroup原理

WaitGroup，可理解为Wait-Goroutine-Group，即等待一组goroutine结束。比如某个goroutine需要等待其他几个goroutine全部完成，那么使用WaitGroup可以轻松实现。

todo

### 原理

WaitGroup的实现原理比较简单。它有一个计数器，初始值为零。当调用Add方法时，它会将计数器加上传入的值。每次调用Done方法时，计数器减去一个值。最后，在调用Wait方法时，程序将阻塞，直到计数器归零。



https://zhuanlan.zhihu.com/p/344973865