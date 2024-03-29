## interface
接口（interface）定义了一个对象的行为规范，只定义规范不实现，由具体的对象来实现规范的细节。

## 一、基础知识
### 1、接口类型
在Go语言中接口（interface）是一种类型，一种抽象的类型。

interface是**一组method的集合**，是duck-type programming的**一种体现**。接口做的事情就像是定义一个协议（规则），只要一台机器有洗衣服和甩干的功能，我就称它为洗衣机。不关心属性（数据），只关心行为（方法）。

牢记：**接口是一种类型，是指针类型。**

### 2、接口的定义

```go
type 接口类型名 interface{
    方法名1( 参数列表1 ) 返回值列表1
    方法名2( 参数列表2 ) 返回值列表2
    …
} 
```
* 接口类型名：使用type将接口定义为自定义的类型名。接口名最好要能突出该接口的类型含义。
* 方法名：当方法名首字母是大写且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包（package）之外的代码访问。
* 参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以省略。

### 2、值接收者和指针接收者实现接口的区别
**指针接收者实现只能以指针方式使用，值接收者都可以。**

* 即使用值接收者实现接口，结构体值类型和结构体指针类型的变量都能存，指针接收者实现接口只能存结构体指针类型的变量。
  
* 如果是按 pointer 调用，go 会自动进行转换，因为有了指针总是能得到指针指向的值是什么。

* 如果是 value 调用，go 将无从得知 value 的原始值是什么，因为 value 是一份拷贝。go 会把指针进行隐式转换得到 value，但反过来则不行。

* 如果接口集中只要有一个方法接收者是指针类型，则必须用 value 调用。todo???

### 2、特性
* 接口是一个或多个方法签名的集合。接口只有方法声明，无实现，无字段。
* 任何类型的方法集中只要拥有该接口'对应的全部方法'签名。就表示它 "实现" 了该接口，无须在该类型上显式声明实现了哪个接口。这称为duck typing。
* 所谓对应方法，是指有相同名称、参数列表 (不包括参数名) 以及返回值。当然，该类型还可以有其他方法。
* 接口可以匿名嵌入其他接口，或嵌入到结构中。
* 接口同样支持匿名字段方法。
* 接口也可实现类似OOP中的多态。
* 空接口可以作为任何类型数据的容器。
* 一个类型可实现多个接口。而接口间彼此独立。
* 接口命名习惯以 er 结尾(Writer,Stringer)。
* **当接口存储的类型和data都为 nil时，接口才为 nil。**
* 如果两个变量类型为接口，则他们必须实现了相同的接口，才能比较。
* **接口不支持直接实例化，只支持赋值：1 将实现接口的对象实例赋值给接口。 2 将一个接口赋值给另一个接口。**
* 特殊接口： stringer  reader   writer
* 一个接口的值（简称接口值）是由一个具体类型和具体类型的值两部分组成的。这两部分分别称为接口的动态类型和动态值。

### 3、内置接口error
```go
//只要实现了Error()函数且返回值为String的都实现了err接口
type error interface { 
    Error()   String
}
```

### 4、接口嵌套
### 5、接口组合(interface 3.go)

## 二、空接口

空接口是指不包含任何方法定义的、空的接口类型。因此任何类型都实现了空接口。

空接口类型的变量可以存储任意类型的变量。
```go
// 定义一个空接口x，注意这里不是用type 
var x interface{} //interface{} 空接口类型， {}表示不包含任何内容的数据结构（或者说数据类型）。
s := "pprof.cn"
x = s
fmt.Printf("type:%T value:%v\n", x, x)
```
### 1、空接口的应用
##### a) 空接口作为函数的参数
使用空接口实现可以接收任意类型的函数参数。**所以interface是所有类型的父类**。

##### b）空接口作为map的值
使用空接口实现可以保存任意值的字典。
```go
var studentInfo = make(map[string]interface{})
studentInfo["name"] = "李白"
studentInfo["age"] = 18
```

### 2、转换为空接口值

```go
interface{}(container)
```

## 三、类型断言

一个接口里面包含的信息：

* **接口类型元数据(静态类型)**

  即原有的接口类型的描述信息。

* **接口值(动态类型)**

### 1、接口值(接口类型变量)

**<font color="red">接口值能够存储所有实现了该接口的实例(即对象)</font>**。

一个接口的值（简称接口值）是由两部分组成:	
* **具体类型，称为接口的动态类型。**
* **具体类型的值，称为接口动态值。**

```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```

![33432424.png](https://pic.imgdb.cn/item/623587c05baa1a80abf19674.png)

**上面接口的接口类型元数据(静态类型)里面还存放了interface的类型元数据，即io.Writer，它里面记录了这个接口类型的描述信息(比如包路径，拥有的Read和Write方法)**。

### 2、类型断言方法

**判断是否是某种动态类型。**

类型断言的关键是明确**接口的动态类型**，以及**对应的类型实现了哪些方法**。而明确这些的关键是类型元数据，以及空接口与非空接口的数据结构。

##### a) 类型判断
```go
x.(T)

x：表示类型为interface{}的变量，即必须为接口类型。
T：表示断言x可能是的类型。
```

该语法返回两个参数，第一个参数是x转化为T类型后的变量，第二个值是一个布尔值，若为true则表示断言成功，为false则表示断言失败。
```go
var x interface{}
x = "pprof.cn"
v, ok := x.(string)
if ok {
    fmt.Println(v) // pprof.cn
} else {
    fmt.Println("类型断言失败")
}

// 或者明确知道了类型, 但是如果类型判断错误会panic
v := x.(string) // 仅接口类型支持
fmt.Println(v) // pprof.cn

// 还有一种是把变量的值转换为空接口值，然后再断言
container := []string{}
value, ok := interface{}(container).([]string) 
fmt.Println(value,ok) // [] true
```

![23114324.png](https://pic.imgdb.cn/item/63197fbc16f2c2beb1f551c1.png)

##### b) switch

```go
switch v := x.(type) {
    case string:
        fmt.Printf("x is a string，value is %v\n", v)
}
```

### 3、接口值转换

```go
var x interface{}
x = "pprof.cn"
v, ok := x.(string)  // 仅接口类型支持
if ok {
    fmt.Println(v)
} else {
    fmt.Println("类型断言失败")
}

// 或者知道了具体类型后，直接转换
v := x.(string) // 仅接口类型支持

// 或者用x.(type)，结合 switch case 判断枚举转换
```

## 四、接口的意义

实际上接口的最大的意义就是**实现多态的思想**，就是我们可以根据interface类型来设计API接口，那么这种API接口的适应能力不仅能适应当下所实现的全部模块，也适应未来实现的模块来进行调用。

调用未来可能就是接口的最大意义所在吧，这也是为什么架构师那么值钱，因为良好的架构师是可以针对interface设计一套框架，在未来许多年却依然适用。

## 五、其他

### 1、显示赋值时，和对变量的赋值不一样

```go
func NewUser() UserModel {
   return User{
      defaultUser: &defaultUser{},
   }
}
func NewUser() UserModel {
   return &User{
      defaultUser: &defaultUser{},
   }
}

// 下面是错误的
func NewUser() *UserModel {
   return &User{
      defaultUser: &defaultUser{},
   }
}
```

## 六、wiki

* Go interface 的 5 个关键点 https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/
* https://www.kancloud.cn/aceld/golang/1958309 todo

