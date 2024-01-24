## 异常处理
Golang 没有结构化异常，使用 panic 抛出错误，recover 捕获错误。

异常的使用场景简单描述：Go中可以抛出一个panic的异常，然后在defer中通过recover捕获这个异常，然后正常处理。

## 一、基础知识
### 1、panic：

**和defer一样，每个G的结构体上也有一个panic的单向链表，越新的panic放在链表头，但打印时，却是从尾部开始打印。**

```go
// runtime/runtime2.go
// 关键结构体
type _panic struct {
    argp      unsafe.Pointer // defer的参数空间地址
    arg       interface{}    // panic 的参数
    link      *_panic        // 链接下一个 panic 结构体
    recovered bool           // 是否恢复，到此为止。设置为true后，除非手动打印，否则不会进入panic打印
    aborted   bool           // the panic was aborted
}
```

![A473-FDAB3F6F8926.png](https://pic.imgdb.cn/item/64f2e2a0661c6c8e54bf613b.png)

* 假如函数F中书写了panic语句，会终止其后要执行的代码，在panic所在函数F内如果存在要执行的defer函数列表，按照defer的逆序执行
* **假如函数G中存在要执行的defer函数列表，按照defer的逆序执行**
* **直到goroutine整个退出，并报告错误**


### 2、recover：

* 用来控制一个goroutine的**panic**行为，捕获panic，从而影响应用的行为

* 一般的调用建议
    a) 在defer函数中，通过recever来终止一个goroutine的panic过程，从而恢复正常代码的执行
    
    `recover()` 并不是说只能在 defer 里面调用，而是**只能在 defer 函数中才能生效**
    
    b) 可以获取通过panic传递的error
    
* **recover不能恢复一些致命错误**，比如

    1、 栈溢出(stack overflow)，内存超限(out of memory)，死锁等

    2、 runtime.throw()的panic都不能恢复，比如并发写map，数组下标超限。

## 二、示例
### 1、基本使用
```go
func test() {
    defer func() {
        if err := recover(); err != nil {
            println(err.(string)) // 将 interface{} 转型为具体类型。因为下面panic传进去的值类型是string类型，所以这里直接转换为string，如果不能确定panic的值类型，则不能这样判断？？todo
            fmt.Println(err) // 这样可以，自动转换为字符串
        }
    }()
    panic("panic error!")
}
// 输出 panic error!
```
由于 panic、recover 参数类型为 interface{}，因此可抛出任何类型对象。

### 2、recover类型判断

```go
defer func() {
   message := recover()
   switch message.(type) {
   case string :
      fmt.Println("string panic :", message)
      break
   case error :
      fmt.Println("errro panic :", message)
      break
   default:
      fmt.Println("unknown panic :", message)
      break
   }
}()
//panic("I am panic ")
//panic(errors.New("I am panic"))
panic(12)
```

## 三、其他
### 1、细节部分

* 利用recover处理panic指令，**defer 必须放在 panic 之前定义，另外 recover 只有在 defer 调用的函数中才有效。**否则无法捕获到panic。
* **recover 处理异常后，逻辑并不会恢复到 panic 那个点去，函数跑到 defer 之后的那个点。**
* panic的参数是nil。这种情况recover捕获后，拿到的返回值也是nil。

### 2、如何区别使用 panic 和 error 两种方式

惯例是导致关键流程出现不可修复性错误的使用 panic，其他使用 error。

## 三、嵌套的情况
### 1、例子1

**先panic的先打印。**

```go
func test() {
   defer func() {
      //if err := recover(); err != nil {
      // println(err.(string)) // 将 interface{} 转型为具体类型。
      //}
      panic("panic error2222")
   }()
   panic("panic error111!")
}
//panic: panic error111!
        panic: panic error2222
```

### 2、例子2

defer里面再panic，会摘除上一个panic

```go
println("=== begin ===")
defer func() { // defer_0
   println("=== come in defer_0 ===")
}()
defer func() { // defer_1
   a := recover()
   fmt.Println(a)
}()
defer func() { // defer_2
   panic("panic 3")
}()
defer func() { // defer_2
   panic("panic 2")
}()
panic("panic 1")
println("=== end ===")

// === begin ===
// panic 3 // recover打印离他最近的panic
// === come in defer_0 ===
```

### 相关wiki

* https://www.bilibili.com/video/BV155411Y7XT笔记 https://www.jianshu.com/p/2da0980108f8

## 四、相关wiki

- 深度细节 | Go 的 panic 和 recover 的真相https://zhuanlan.zhihu.com/p/418257257 