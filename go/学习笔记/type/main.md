## type

## 一、内置类型
比如int8，int16，string，slice，map等。

## 二、自定义类型

使用type定义的类型，都是自定义类型。这种方式也可以被叫做对类型的再定义。

```go
type MyString2 string // 注意，这里没有等号。
```

`MyString2`和`string`就是两个不同的类型了。这里的`MyString2`是一个新的类型，不同于其他任何类型。

对于这里的类型再定义来说，`string`可以被称为`MyString2`的潜在类型。潜在类型的含义是，某个类型在本质上是哪个类型。

潜在类型相同的不同类型的值之间是可以进行类型转换的。因此，`MyString2`类型的值与`string`类型的值可以使用类型转换表达式进行互转。

但对于集合类的类型`[]MyString2`与`[]string`来说这样做却是不合法的，因为`[]MyString2`与`[]string`的潜在类型不同，分别是`[]MyString2`和`[]string`。另外，即使两个不同类型的潜在类型相同，它们的值之间也不能进行判等或比较，它们的变量之间也不能赋值。

## 三、类型别名

**别名类型与其源类型的区别只是在名称上，它们是完全相同的。**

我们可以用关键字`type`声明自定义的各种类型。这些类型必须在 Go 语言基本类型和高级类型的范畴之内。在它们当中，有一种被叫做“别名类型”的类型。

```go
type MyString = string
```

`MyString`是`string`类型的别名类型。

源类型与别名类型是一对概念，是两个对立的称呼。别名类型主要是为了代码重构而存在的。 

Go 语言内建的基本类型中就存在两个别名类型。`byte`是`uint8`的别名类型，而`rune`是`int32`的别名类型。

### 1、类型别名在switch中

既然类型别名和原类型是相同的，那么在`switch - type中，你不能将原类型和类型别名作为两个分支，因为这是重复的case:

```go
type D = int
var v interface{} 
var d D = 100
v = d
switch i := v.(type) {
	case int:
		fmt.Println("it is an int:", i)
	// case D:
	// 	fmt.Println("it is D type:", i)
}
```

### 2、类型循环

类型别名在定义的时候不允许出现循环定义别名的情况，如下面所示：

```go
type T1 = T2
type T2 = T1

type T1 = struct {
	next *T2
}
type T2 = T1
```

### 3、可导出性

如果定义的类型别名是`exported` (首字母大写)的，那么别的包中就可以使用，它和原始类型是否可`exported`没关系。也就是说，你可以为`unexported`类型定义一个`exported`的类型别名。

```go
type t1 struct {
	S string
}

type T2 = t1
```

### 4、方法集

既然类型别名和原始类型是相同的，那么它们的方法集也是相同的。

```go
type T1 struct{}
type T3 = T1
func (t1 T1) say()       {}
func (t3 *T3) greeting() {}
//func (t1 *T1) greeting() {} // 类型别名和原始类型定义了相同的方法，编译报错，因为有重复的方法定义。
func main() {
	var t1 T1
	var t3 T3
	t1.say()
	t1.greeting()
	t3.say()
	t3.greeting()
}
```

### 5、embedded type

`s.say｀不知道该调用`s.T1.say`还是`s.T3.say`，所以这个时候你需要明确的调用。

```go
type T1 struct{}
type T3 = T1
func (t T1) say(){}
type S struct {
   T1
   T3
}
func main() {
   var s S
   s.say() //  ambiguous selector s.say
   s.T1.say() // 正常
}
```

进一步想，这样是不是我们可以为其它库中的类型增加新的方法了， 比如为标准库的`time.Time`增加一个滴答方法:

```go
type NTime = time.Time
func (t NTime) Dida() {
	fmt.Println("嘀嗒嘀嗒嘀嗒嘀嗒搜索")
}
func main() {
	t := time.Now()	
	t.Dida() // cannot define new methods on non-local type time.Time
}
```

## 三、自定义类型和类型别名区别

类型别名意味着这两种类型的数据可以互相赋值，而类型定义要和原始类型赋值的时候需要类型转换(Conversion `T(x)`)。

![3453453456.png](https://pic.imgdb.cn/item/6319afae16f2c2beb1309fe3.png)





## 四、类型别名作用 todo

官方解释：

1. 在大规模的重构项目代码的时候，尤其是将一个类型从一个包移动到另一个包中的时候，有些代码使用新包中的类型，有些代码使用旧包中的类型， 比如`context`。
2. 允许一个庞大的包分解成内部的几个小包，但是小包中的类型需要集中暴漏在上层的大包中。

通俗的解释

1. 名字能够取的更通俗易懂；
2. 须要修改数据类型时，只用改定义的那一处地方；
3. 能够很方便的添加特有方法，以实现某些接口。

对于大型的代码库来讲，可以重构其总体结构是很是重要的，包括修改某些 API 所属的包。大型重构应该支持一个过渡期：从旧位置和新位置得到的 API 都应该是可用的，并且能够混合使用这些 API 的引用。Go 已经为常量、函数或变量的重构提供了可行的机制，可是并不支持类型。类型别名提供了一种机制，它可使得 oldpkg.OldType 和 newpkg.NewType 是相同的，而且引用旧名称的代码与引用新名称的代码能够互相操做。数据类型

考虑将一个类型从一个包移动到另外一个包中的状况，好比从 oldpkg.OldType 到 newpkg.NewType。能够在包 oldpkg 中指定一个新类型的别名 type OldType = newpkg.NewType，这样之前的代码都无需修改。引

