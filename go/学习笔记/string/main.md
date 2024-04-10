## string

字符串的底层数据结构:
```go
type StringHeader struct {
    Data uintptr
    Len  int
}
```
### 1、byte和rune类型
go的字符有以下两种
* uint8类型，或者叫 byte 型，代表了ASCII码的一个字符。
* rune类型，代表一个 UTF-8字符。
  

当需要处理中文、日文或者其他复合字符时，则需要用到rune类型。rune类型实际是一个int32。 Go 使用了特殊的 rune 类型来处理 Unicode，让基于 Unicode的文本处理更为方便，也可以使用 byte 型进行默认字符串处理，性能和扩展性都有照顾。

### 2、读取字符串中某个字符
``` go
a := "abc"
a[1] = 'd' // panic
fmt.Println(string(a[1]))
```
**字符串中某个字符可读不可写。**

### 3、遍历字符串
```go
func traversalString() {
	s := "pprof.cn博客"
	for i := 0; i < len(s); i++ { //byte
		fmt.Printf("%v(%c) ", s[i], s[i])
	}
	fmt.Println()
	for _, r := range s { //rune
		fmt.Printf("%v(%c) ", r, r)
	}
	fmt.Println()
}
```
输出：
```
112(p) 112(p) 114(r) 111(o) 102(f) 46(.) 99(c) 110(n) 229(å) 141() 154() 229(å) 174(®) 162(¢)
112(p) 112(p) 114(r) 111(o) 102(f) 46(.) 99(c) 110(n) 21338(博) 23458(客)
```

因为UTF8编码下一个中文汉字由3~4个字节组成，所以我们不能简单的按照字节去遍历一个包含中文的字符串，否则就会出现上面输出中第一行的结果。

### 4、特性
* **字符串是字节的定长数组。字符串底层是一个byte数组，所以可以和[]byte类型相互转换。**
* 字符串是由byte字节组成，所以字符串的长度是byte字节的长度。 
* rune类型用来表示utf8字符，一个rune字符由一个或多个byte组成。
* 字符串的内部实现使用UTF-8编码，占1-4个字节(减少了内存和硬盘的占用)

### 5、修改字符串

* **字符串**是不能修改的。**需要先将其转换成[]rune或[]byte，完成后再转换为string。**
* **无论哪种转换，都会重新分配内存，并复制字节数组。**因为go的编译器会把字符串内容分配到只读内存段，所以不能直接修改。

```go
func changeString() {
	s1 := "hello"
	// 强制类型转换
	byteS1 := []byte (s1)//这里要用括号括起来
	byteS1[0] = 'H'
	fmt.Println(string(byteS1))

	s2 := "博客"
	runeS2 := []rune(s2)
	runeS2[0] = '狗'
	fmt.Println(string(runeS2))
}
```

所以，string转换为[]byte是会拷贝内存的，因为string和[]byte在内存布局上是不同的

- string是不可变的字节序列，内部包含字节数组指针和长度信息
- []byte是可变字节序列，保存了底层字节数组指针和容量信息

### 6、字符串转义 ？？？

```go
strconv.Quote(str)
strconv.Unquote(str)

escapedString := "\"HelloWorld\""
escapedString,_:= strconv.Unquote(str)

escapedString := "\"Hello\"World\""
escapedString,_:= strconv.Unquote(str) // 报错

// 对于前端提交上来的json字符串，比如
// {\"extAppid\":\"wx8f268909830e94a4\",\"extEnable\":true}
// 也可尝试用strconv.Unquote()转换下

```