## function value

函数，在GO语言中属于头等对象，可以被当作参数传递、也可以作为函数返回值、绑定到变量。Go语言称这样的参数、返回值和变量为“Function Value”。

将一个函数赋值给一个变量，则这个变量叫做函数表达式。

方法也是同样的道理，**最终都以方法表达式执行**。

```go
func (p P) get() {}
p := P{}

f1 := P.get
f1(p) // 方法表达式

f := p.get // 方法变量
f() // 底层会默认转换为方法表达式，即P.get(f)
```

本质上来说，函数表达式，方法表达式，方法变量，都是Function Value。

## 一、底层原理

Function Value本质上是一个指针，却不直接指向函数指令入口，而是指向runtime.funcval结构体。

```go
type funcval struct {
    fn uintptr
}
```

这个结构体从定义上看只有一个地址，这个地址才是函数的指令入口。所以，一个Function Value是以下图所示形式存在的。

![640.png](https://pic.imgdb.cn/item/623fc83e27f86abb2a9a3110.png)


* Go语言里Function Value本质上是指向funcval结构体的指针；
* Go语言里闭包只是拥有捕获列表的Function Value；
* 捕获变量在外层函数与闭包函数中要保持一致。


### todo
https://mp.weixin.qq.com/s/iFYkcLbNK5pOA37N7ToJ5Q