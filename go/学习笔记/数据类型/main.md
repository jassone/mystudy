## 数据类型
## 一、内建变量类型
### 1、bool
- 布尔类型变量的默认值为false。
- 布尔型无法参与数值运算，也无法与其他类型进行互转。

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

### 1、打印数据类型等

```go
//打印数据类型
fmt.Printf("%T", s1) 
fmt.Println(reflect.TypeOf(arr2))

//打印指针地址
fmt.Printf("ptr的指针地址为: %p\n", ptr)  
```

### 2、int 和 int64的区别

int在32位系统上是4个字节
int在64位系统上是8个字节
int32在哪都是4字节
int64在哪都是8字节

### 3、nil

nil 可以用作 interface、function、pointer、map、slice 和 channel 的“空值”。

## 四、wiki

* https://studygolang.com/articles/16011?fr=sidebar