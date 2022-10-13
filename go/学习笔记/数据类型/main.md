## 数据类型
## 一、内建变量类型
### 1、bool
- 布尔类型变量的默认值为false。
- 不允许将整型强制转换为布尔型.
- 布尔型无法参与数值运算，也无法与其他类型进行转换。

### 2、字符串
Go语言中的字符串**以原生数据类型出现**，使用字符串就像使用其他原生数据类型（int、bool、float32、float64 等）一样。 Go 语言里的字符串的**内部实现使用UTF-8编码**。

### 3、整型 
(u)int（32位或64位）,(u)int8,(u)int16,(u)int32,(u)int64,uintptr
byte = uint8
rune = int32 (rune类型实际是一个int32，可以看成是int32的一个别名，但是是两种类型)

##### 注意范围
int8 -128-127

### 3、浮点数
float32,float64

### 4、复数
complex64,complex128

## 二、无类型常量
无类型的布尔型，无类型的整数，无类型的字符，无类型的浮点数，无类型的复数，无类型的字符串
下面是延迟明确常量的具体类型
```go
var a  float32 = math.Pi 
var b  float64 = math.Pi
```

## 三、细节
### 1、go 只有强制类型转换，没有隐式类型转换,如
```go
// T(表达式) T表示要转换的类型。表达式包括变量、复杂算子和函数返回值等.
int64(1000)

// 注意这两个不一样
fmt.Println(int('0')) //48
fmt.Println(strconv.Atoi("0")) // 0
```

### 2、打印数据类型等
```go
//打印数据类型
fmt.Printf("%T", s1) 
fmt.Println(reflect.TypeOf(arr2))

//打印指针地址
fmt.Printf("ptr的指针地址为: %p\n", ptr)  
```

### 3、int 和 int64的区别 
int在32位系统上是4个字节
int在64位系统上是8个字节
int32在哪都是4字节
int64在哪都是8字节

### 4、nil
nil 可以用作 interface、function、pointer、map、slice 和 channel 的“空值”。

### 5、类型转换  todo

- 当整数值的类型的有效范围由宽变窄时，只需在补码形式下截掉一定数量的高位二进制数即可。
- 当把一个浮点数类型的值转换为整数类型值时，前者的小数部分会被全部截掉。

###### a) 数字转字符串

**虽然直接把一个整数值转换为一个`string`类型的值是可行的，但值得关注的是，被转换的整数值应该可以代表一个有效的 Unicode 代码点，否则转换的结果将会是`"�"`（仅由高亮的问号组成的字符串值）。**

```go
string(97) // 得到 a。
string(20005) // 得到 严。 20005为严的unicode的十进制编码值
string(12133) // 得到 ⽥。
```

```go
string(-1) // 得到�, 因为-1肯定无法代表一个有效的 Unicode 代码点。
strconv.FormatInt(int64,10) // 正确方式
```

##### b）**`string`类型与各种切片类型之间的互转**

- 首先一个值在从`string`类型向`[]byte`类型转换时代表着以 UTF-8 编码的字符串会被拆分成零散、独立的字节。
- 所以反过来，除了与 ASCII 编码兼容的那部分字符集，以 UTF-8 编码的某个单一字节是无法代表一个字符的。

```go
string([]byte{'\xe4', '\xbd', '\xa0', '\xe5', '\xa5', '\xbd'}) // 你好
```

- 一个值在从`string`类型向`[]rune`类型转换时代表着字符串会被拆分成一个个 Unicode 字符。
- 反过 Unicode 字符转换为字符串

```go
string([]rune{'\u4F60', '\u597D'}) // 你好
```

## 五、wiki

* https://studygolang.com/articles/16011?fr=sidebar