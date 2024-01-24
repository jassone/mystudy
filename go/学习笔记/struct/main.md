## struct
## 一、结构体概念
结构体也是自定义类型的一种(参考‘自定义类型和类型别名’笔记)。

golang中是没有class的,但是有一个结构体struct,有点类似,他没有继承的概念,但是他有一个类似功能的嵌入结构体可以封装多个基本数据类型，也就是我们可以通过struct来定义自己的类型了。

struct代表若干字段的集合，有时将多个数据看做一个整体要比单独使用这些数据更有意义，在这种情况下就适合使用结构体。

## 二、基础知识
### 1、结构体的定义
使用type和struct关键字来定义结构体
```go
type 类型名 struct {
    字段名 字段类型
    字段名 字段类型
    …
}
// 定义
type s struct{}

// 定义并赋值(空)
s :=struct{}{}
```

### 2、匿名结构体

```go
s := struct {
    // 匿名结构体字段定义
    Field1 Field1Type
    Field2 Field2Type
}{
    // 字段值初始化
    Field1: Value1,
    Field2: Value2,
}
```

### 3、创建指针类型结构体

**通过使用new关键字对结构体进行实例化，得到的是结构体的指针地址**。 

```go
var p2 = new(person)
fmt.Printf("%T\n", p2)     //*main.person
fmt.Printf("p2=%#v\n", p2) //p2=&main.person{name:"", city:"", age:0}
```
从打印的结果中我们可以看出p2是一个结构体指针。

### 4、取结构体的实例化地址

**使用&对结构体进行取地址操作相当于对该结构体类型进行了一次new实例化操作。**
```go
p3 := &person{}
fmt.Printf("%T\n", p3)     //*main.person
fmt.Printf("p3=%#v\n", p3) //p3=&main.person{name:"", city:"", age:0}
p3.name = "博客"
```
p3.name = "博客"其实在底层是(*p3).name = "博客"，这是Go语言帮我们实现的语法糖。

### 5、结构体初始化/实例化

只有当结构体实例化时，才会真正地分配内存。也就是必须实例化后才能使用结构体的字段。

**结构体本身也是一种类型，我们可以像声明内置类型一样使用var关键字声明结构体类型。**

```go
var 结构体实例 结构体类型
// 如
var p4 person
```
##### 使用键值对初始化
```go
p5 := person{
    name: "pprof.cn",
}
```
当某些字段没有初始值的时候，该字段可以不写，该字段类型的值是零值。
##### 结构体指针进行键值对初始化

```go
p6 := &person{
    name: "pprof.cn",
}
```
##### 使用值的列表初始化
初始化的时候不写键，直接写值。
```go
p8 := &person{
    "pprof.cn",
}
```

### 6、构造函数
Go语言的结构体没有构造函数，我们可以自己实现(通常以new开头，比较chang常用)。 

例如，下方的代码就实现了一个person的构造函数。 因为struct是值类型，如果结构体比较复杂的话，值拷贝性能开销会比较大，所以该构造函数返回的是结构体指针类型。

```go
func newPerson(name, city string, age int8) *person {
    return &person{
        name: name,
        city: city,
        age:  age,
    }
}
//调用构造函数

p9 := newPerson("pprof.cn", "测试", 90)
fmt.Printf("%#v\n", p9)
```

### 7、结构体内存布局

```go
type test struct {
    a int8
    b int8
    c int8
}
n := test{
    1, 2, 3,
}
fmt.Printf("n.a %p\n", &n.a)
fmt.Printf("n.b %p\n", &n.b)
fmt.Printf("n.c %p\n", &n.c)
n.a 0xc0000b2002
n.b 0xc0000b2003
n.c 0xc0000b2004
```

### 8、任意类型添加方法
```go
//MyInt 将int定义为自定义MyInt类型
type MyInt int

//SayHello 为MyInt添加一个SayHello的方法
func (m MyInt) SayHello() {
    fmt.Println("Hello, 我是一个int。")
}
func main() {
    var m1 MyInt
    m1.SayHello() //Hello, 
}
```
### 9、结构体的匿名字段
结构体允许其成员字段在声明时没有字段名而只有类型，这种没有名字的字段就称为匿名字段(也称为嵌入字段)。

```go
type Student struct{
    name string
}
type Student2 struct{
    name string
}
type Person struct {
    string
    int
    Student // 匿名字段(也称为嵌入字段)
    *Student2 // 指针类型匿名字段
}
func main() {
    p1 := Person{
        "pprof.cn",
        18,
        Student{"name1"},
        &Student2{"name2"},
    }
}
```

### 10、嵌套结构体
### 11、嵌套匿名结构体
### 12、嵌套结构体的字段名冲突
嵌套结构体内部可能存在相同的字段名。这个时候为了避免歧义需要指定具体的内嵌结构体的字段。
```
user3.Address.CreateTime = "2000" //指定Address结构体中的CreateTime
user3.Email.CreateTime = "2000"   //指定Email结构体中的CreateTime
```

### 13、结构体的“继承”
Go语言中使用结构体也可以实现其他编程语言中面向对象的继承。

### 14、结构体字段的可见性
结构体中字段大写开头表示可公开访问，小写表示私有（仅在定义当前结构体的包中可访问）。

### 15、空结构体

空结构不申请空间，无性能消耗

```go
a := struct {
}{}
b := struct {
}{}
fmt.Printf("%p\n", &a) // 二者指针地址一样
fmt.Printf("%p\n", &b)
```

## 三、方法和接收者
Go语言中的方法（Method）是一种作用于特定类型变量的函数。**这种特定类型变量叫做接收者（Receiver）**。接收者的概念就类似于其他语言中的this或者 self。

方法的定义格式如下：
```go
func (接收者变量 接收者类型) 方法名(参数列表) (返回参数) {
    函数体
}
```
* 接收者变量：接收者中的参数变量名在命名时，官方建议使用接收者类型名的第一个小写字母，而不是self、this之类的命名。例如，Person类型的接收者变量应该命名为 p，Connector类型的接收者变量应该命名为c等。
* 接收者类型：接收者类型和参数类似，可以是指针类型和非指针类型。
* 方法名、参数列表、返回参数：具体格式与函数定义相同。

### 1、指针类型的接收者
将指针值复制一份传入，修改后改变接收者
```go
func (p *Person) SetAge(newAge int8) {
    p.age = newAge
}
```
### 2、值类型的接收者
将接收者的值复制一份
```go
func (p Person) SetAge(newAge int8) {
    p.age = newAge
}
```

### 3、什么时候应该使用指针类型接收者
* 需要修改接收者中的值
* 接收者是拷贝代价比较大的大对象
* 保证一致性，如果有某个方法使用了指针接收者，那么其他的方法也应该使用指针接收者。

## 四、细节部分

### 1、特性
* golang中严格意义上是没有oop的思想，只是通过结构体的方式来实现了面向对象
* golang中没有类，通过结构体可以实现同等的地位
* golang中去除了传统的oop语法，继承，重载，构造，析构，隐藏this的特性
* golang仍然有面向对象三大特性，只是实现和面向对象方法有所不同，没有extends关键字
* 结构体是非常灵活的，耦合度低

### 3、其他
* 结构体是值类型
* 字段类型也可以是结构体，甚至是字段所在结构体的类型(比如递归循环)
* 名字一般使用CamelCase(驼峰命名法)
* 结构创建在堆上还是栈上？ 不需要知道，解析器自己决定
* 一致性：如果指针接受者，最好都是指针接受者
* struct 的变量字段不能使用 := 来赋值。

