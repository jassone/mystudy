## 方法
**Golang 方法总是绑定对象实例，并隐式将实例作为第一实参 (receiver)。**

一个方法就是一个包含了接受者的函数，接受者可以是命名类型或者结构体类型的一个值或者是一个指针。

## 一、基础知识

### 1、方法定义
```go
func (recevier type) methodName(参数列表)(返回值列表){}

参数和返回值可以省略
```
### 2、基本特性
* 只能为当前包内命名类型定义方法。
* 参数 receiver 可任意命名。如方法中未曾使用 ，可省略参数名。
* 参数 receiver 类型可以是 T 或 \*T。**基类型 T 不能是接口或指针。**
* 不支持方法重载，receiver 只是参数签名的组成部分。
* 可用实例 value 或 pointer 调用全部方法，编译器自动转换。

### 3、注意点
**接受者可以拿到本身的所有数据**。

是修改源数据还是修改副本**依据接收者定义是否是指针**。

* 接收者为值类型时，传值传引都可以
* 接收者为指针类型时，传值传引都可以

### 4、普通函数与方法的区别---重要
* **对于普通函数，接收者(其实是参数)为值类型时，不能将指针类型的数据直接传递，反之亦然。**
* **对于方法（如struct的方法），接收者为值类型时，可以直接用指针类型的变量调用方法，反过来同样也可以。**

### 5、匿名字段
Golang匿名字段 ：可以像字段成员那样访问匿名字段方法，编译器负责查找。

## 二、方法集
Golang方法集 ：每个类型都有与之关联的方法集，这会影响到接口实现规则。

* 类型 T 方法集包含全部 receiver T 方法。类型 \*T 方法集包含全部 receiver T + \*T 方法。todo
* 如类型 S 包含匿名字段 T，则 S 和 *S 方法集包含 T 方法。 
* 如类型 S 包含匿名字段 \*T，则 S 和 *S 方法集包含 T + \*T 方法。 
* 不管嵌入 T 或 \*T，\*S 方法集总是包含 T + *T 方法。
* 用实例 value 和 pointer 调用方法 (含匿名字段) 不受方法集约束，编译器总是查找全部方法，并自动转换 receiver 实参。
* 不允许为T和*T定义同名方法(详见包装方法知识)。

## 三、表达式
Golang 表达式 ：根据调用者不同，方法分为两种表现形式:
```go
instance.method(args...)
<type>.func(instance, args...)
```
前者称为 method value，后者 method expression。

两者都可像普通函数那样赋值和传参，区别在于 **method value 绑定实例**，而 **method expression 则须显式传参**。

```go
type User struct {
    id   int
    name string
}

func (self *User) Test() {
    fmt.Printf("%p, %v\n", self, self)
}

func main() {
    u := User{1, "Tom"}
    u.Test()

    mValue := u.Test
    mValue() // 隐式传递 receiver

    mExpression := (*User).Test
    mExpression(&u) // 显式传递 receiver
}
输出结果都是   0xc42000a060, &{1 Tom}
```

## 四、自定义error
详见项目笔记