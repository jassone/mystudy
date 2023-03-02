## 异常处理
Golang 没有结构化异常，使用 panic 抛出错误，recover 捕获错误。

异常的使用场景简单描述：Go中可以抛出一个panic的异常，然后在defer中通过recover捕获这个异常，然后正常处理。

## 一、基础知识
### 1、panic：

* 假如函数F中书写了panic语句，会终止其后要执行的代码，在panic所在函数F内如果存在要执行的defer函数列表，按照defer的逆序执行
* **返回函数F的调用者G，在G中，调用函数F语句之后的代码不会执行，假如函数G中存在要执行的defer函数列表，按照defer的逆序执行**
* **直到goroutine整个退出，并报告错误**


### 2、recover：

* 用来控制一个goroutine的**panic**行为，捕获panic，从而影响应用的行为
* 一般的调用建议
    a) 在defer函数中，通过recever来终止一个goroutine的panic过程，从而恢复正常代码的执行
    b) 可以获取通过panic传递的error

## 二、示例
### 1、基本使用
```go
func test() {
    defer func() {
        if err := recover(); err != nil {
            println(err.(string)) // 将 interface{} 转型为具体类型。
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
### 1、注意
* 利用recover处理panic指令，**defer 必须放在 panic 之前定义，另外 recover 只有在 defer 调用的函数中才有效。**否则无法捕获到panic。
* **recover 处理异常后，逻辑并不会恢复到 panic 那个点去，函数跑到 defer 之后的那个点。**

### 2、如何区别使用 panic 和 error 两种方式
惯例是导致关键流程出现不可修复性错误的使用 panic，其他使用 error。

## 三、嵌套的情况
先panic的先打印。

todo
* https://www.bilibili.com/video/BV155411Y7XT?spm_id_from=333.999.0.0