## duck typing
一般来讲，使用 duck typing 的编程语言往往被归类到“动态类型语言”或者“解释型语言”里，比如 Python, Javascript, Ruby 等等；而其它的类型系统往往被归到“静态类型语言“中，比如 C/C++/Java。

Go 的类型系统采取了折中的办法。

### 1、静态类型系统
一个类型不需要显式地声明它实现了某个接口，但仅当某个变量的类型实现了某个接口的方法，这个变量才能用在要求这个接口的地方。

```go
type ISayHello interface {
    SayHello()
}

type Person struct {}

func (person Person) SayHello() {
    fmt.Printf("Hello!")
}

type Duck struct {}

func (duck Duck) SayHello() {
    fmt.Printf("ga ga ga!")
}

func greeting(i ISayHello) {
    i.SayHello()
}

func main () {
    person := Person{}
    duck   := Duck{}
    var i ISayHello
    i = person
    greeting(i)
    i = duck
    greeting(i)
}
// 最后输出： Hello! ga ga ga
```
两种类型 Duck 和 Person 都实现了sayHello 这一方法。

函数 greeting 要求一个实现了 sayHello 方法的变量。这个变量与一般变量不同，称为“接口变量”。 如果某个变量 t 的类型 T 实现了某个接口 I 所要求的所有方法，那么这个变量 t 就能被赋值给 I 的接口变量 I。调用 I 的方法，最终就是调用 t 的方法。

### 2、为什么说这是一种折中的方法：
第一，类型 T 不需要显式地声明它实现了接口 I。只要类型 T 实现了所有接口 I 规定的函数，它就自动地实现了接口 I。 这样就像动态语言一样省了很多代码，少了许多限制。

第二，**在把 duck 或者 person 传递给 greeting 前，需要显式或者隐式地把它们转换为接口 I 的接口变量 i。这样就可以和其它静态类型语言一样，在编译时检查参数的合法性。正是因为“接口变量”这一类型的存在，Golang 实现了它独特的 “易用” 与 “安全” 二者兼得的多态机制。**“不需要声明实现接口”，这样就省去了很多代码。但是 C语言的 GObject 库里，要声明一个类实现了某个接口，需要写不少规定的代码。同时，转换为 接口变量这一过程是在编译时就完成的，因此，可以在编译时就找出动态语言里在运行时才能发现的代码错误。

### 3、标准库中使用
在 Golang 的 standard library中，这一特性被使用得淋漓尽致。

比如，用 fmt.Fprintf 向一个 http 连接写入 http 响应：

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

Golang 的 fmt.Fprintf 函数的第一个参数的类型是一个 io.Writer 接口的接口变量。
type Writer interface {
    Write(p []byte) (n int, err error)
}
```
