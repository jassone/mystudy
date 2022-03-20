## string

### 1、byte和rune类型
go的字符有以下两种
* uint8类型，或者叫 byte 型，代表了ASCII码的一个字符。
* rune类型，代表一个 UTF-8字符。
  
当需要处理中文、日文或者其他复合字符时，则需要用到rune类型。rune类型实际是一个int32。 Go 使用了特殊的 rune 类型来处理 Unicode，让基于 Unicode的文本处理更为方便，也可以使用 byte 型进行默认字符串处理，性能和扩展性都有照顾。

### 2、遍历字符串
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

### 3、特性
* **字符串是字节的定长数组。字符串底层是一个byte数组，所以可以和[]byte类型相互转换。**
* 字符串是不能修改的。
* 字符串是由byte字节组成，所以字符串的长度是byte字节的长度。 
* rune类型用来表示utf8字符，一个rune字符由一个或多个byte组成。
* 字符串的内部实现使用UTF-8编码，占1-4个字节(减少了内存和硬盘的占用)
* 字符串的值为双引号(")中的内容
* 字符拼接用+,  对于大数量的字符串，bytes.Buffer拼接字符串销量更高

### 4、修改字符串
* **需要先将其转换成[]rune或[]byte，完成后再转换为string。**
* **无论哪种转换，都会重新分配内存，并复制字节数组。**

```go
func changeString() {
	s1 := "hello"
	// 强制类型转换
	byteS1 := []byte(s1)
	byteS1[0] = 'H'
	fmt.Println(string(byteS1))

	s2 := "博客"
	runeS2 := []rune(s2)
	runeS2[0] = '狗'
	fmt.Println(string(runeS2))
}
```