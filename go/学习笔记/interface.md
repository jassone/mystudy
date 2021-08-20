https://studygolang.com/articles/3012

概念
* 只要某个类型拥有该接口的所有方法签名，即算实现该接口，无需显式声明实现哪个接口
* 接口只有方法声明，无实现，无字段
* 当接口存储的类型和对象都为 nil 时，接口才为 nil
* 接口调用不会做 receiver 的自动转换，即指针只能传指针 ？？
* 空接口可以作为任何类型数据的容器
* 接口变量里面有：实现者的类型，实现者的指针（指针指向实现者）  ？？？
* 接口变量自带指针？？？
* 接口变量同样采用值传递，几乎不需要使用接口的指针？？
* 如果两个变量类型为接口，则他们必须实现了相同的接口，才能比较
* 接口本质上是一种类型，是指针类型
* 接口不支持直接实例化，只支持赋值：1将实现接口的对象实例赋值给接口 2 将一根接口赋值给另一个接口


为了保护你的Go语言职业生涯，请牢记接口（interface）是一种类型。
interface变量存储的是实现者的值(比如结构体的值)，的变量中存储的是实现了 interface 的类型的对象值，这种能力是 duck typing 


特殊接口： stringer  reader   writer

理解 Go interface 的 5 个关键点
https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/


2 空接口是指没有定义任何方法的接口。因此任何类型都实现了空接口。
空接口类型的变量可以存储任意类型的变量。

3 一个接口的值（简称接口值）是由一个具体类型和具体类型的值两部分组成的。这两部分分别称为接口的动态类型和动态值。

4 两种声明
func download(r Retriever2)  string {
   return r.Get("http://www.imooc.com")
}
// 显式声明
var re Retriever2
re = mock.Retriever{Contents:"baidu"}

fmt.Println(download(re))

// 简写，隐式声明
fmt.Println(download(mock.Retriever{Contents:"this is a fake imook.com"})) // 这里只要包含了接口的方法即可

5 指针接收者实现只能以指针方式使用，值接收者都可以
(即：使用值接收者实现接口，结构体值类型和结构体指针类型的变量都能存，指针接收者实现接口只能存结构体指针类型的变量)
如果是按 pointer 调用，go 会自动进行转换，因为有了指针总是能得到指针指向的值是什么。
如果是 value 调用，go 将无从得知 value 的原始值是什么，因为 value 是份拷贝。go 会把指针进行隐式转换得到 value，但反过来则不行。
func download(r Rever)  string { // 值接收
   return r.Get("http://www.imooc.com")
}
var r Rever
//r = mock.Retriever2{"this is a fake imook.com"} //  传值
r = &mock.Retriever2{"this is a fake imook.com"}  // 传指针
fmt.Println(download(r))

demo
func (d *dog) move() {
    fmt.Println("狗会动")
}
func main() {
    var x Mover
    var wangcai = dog{} // 旺财是dog类型
    x = wangcai         // x不可以接收dog类型
    var fugui = &dog{}  // 富贵是*dog类型
    x = fugui           // x可以接收*dog类型
}

此时实现Mover接口的是*dog类型，所以不能给x传入dog类型的wangcai，此时x只能存储*dog类型的值。

空接口是指没有定义任何方法的接口。因此任何类型都实现了空接口。
空接口类型的变量可以存储任意类型的变量。
func main() {
    // 定义一个空接口x
    var x interface{}
    s := "pprof.cn"
    x = s
    fmt.Printf("type:%T value:%v\n", x, x)
    i := 100
    x = i
    fmt.Printf("type:%T value:%v\n", x, x)
    b := true
    x = b
    fmt.Printf("type:%T value:%v\n", x, x)
}
空接口作用
1 使用空接口实现可以接收任意类型的函数参数。
// 空接口作为函数参数
func show(a interface{}) {
    fmt.Printf("type:%T value:%v\n", a, a)
}

虽然函数的参数可以接受任何类型，并不表示 v 就是任何类型，在函数 doSomething 内部 v 仅仅是一个 interface 类型，之所以函数可以接受任何类型是在 go 执行时传递到函数的任何类型都被自动转换成 interface{}

2 使用空接口实现可以保存任意值的字典。
// 空接口作为map值
    var studentInfo = make(map[string]interface{})
    studentInfo["name"] = "李白"
    studentInfo["age"] = 18
    studentInfo["married"] = false
    fmt.Println(studentInfo)


6 断言(只有接口才有断言)
一个接口的值（简称接口值）是由一个具体类型和具体类型的值两部分组成的。这两部分分别称为接口的动态类型和动态值。

func justifyType(x interface{}) {
    switch v := x.(type) {
    case string:
        fmt.Printf("x is a string，value is %v\n", v)
    default:
        fmt.Println("unsupport type！")
    }
}
x.(type) 只能用在switch中

断言的几种判断方式
var r Rever
r = &mock.Retriever2{"this is a fake imook.com"}  
// Type assertion
if mockRetriever, ok := r.(*mock.Retriever2 ); ok {} // 注意这里是指针类型

r = mock.Retriever2{"this is a fake imook.com"}
// Type assertion
if mockRetriever, ok := r.(mock.Retriever2 ); ok {}

if res,ok := I.([]int);ok {
    // 判断I接口是否是[]int类型
}

使用switch
    switch v := r.(type) {
	case mock.Retriever2:
		fmt.Println("Contents:", v.Contents)
	case *real.Retriever:
		fmt.Println("UserAgent:", v.UserAgent)
	}
* 
7 接口组合(interface 3.go)
