## go字符串拼接性能问题

## 一、方法汇总

### 1、直接相加

这种方法的缺点是效率低下，因为每次拼接都会创建一个新的字符串，分配新的内存空间，并复制原来的字符串内容。如果需要拼接很多字符串，或者在循环中进行拼接，这种方法会造成大量的内存开销和性能损失。

当使用 `+` 拼接 2 个字符串时，生成一个新的字符串，那么就需要开辟一段新的空间，新空间的大小是原来两个字符串的大小之和。

### 2、fmt.Sprintf

```go
s1 := "Hello"
s2 := "World"
s3 := fmt.Sprintf("%s %s", s1, s2) // s3 = "Hello World"
```

这种方法的缺点是效率低下，因为它涉及到反射、格式解析、内存分配等操作。而且，这种方法不适合在不知道参数数量和类型的情况下进行拼接。

### 3、fmt.Sprint()拼接

```go
res := ""
res = fmt.Sprint(res, "string")
```

### 4、append []byte

```go
s1 := "Hello"
s2 := "World"
b := []byte(s1)
b = append(b, ' ')
b = append(b, s2...)
s3 := string(b) // s3 = "Hello World"
```

如果长度是可预知的，那么创建 `[]byte` 时，我们还可以预分配切片的容量(cap)。

这种方法的优点是效率高，因为它直接操作字节，避免了额外的内存分配和复制。但是，这种方法的缺点是不够简洁和易用，需要进行多次类型转换。

### 5、strings.Join



### 5、bytes.Buffer写入

```go
 var res bytes.Buffer
    for i := 0; i < n; i++ {
        res.WriteString("string")
     }
 return res.String()
```

### 4、strings.Builder写入

生成器用于使用Write方法高效地生成字符串。它最大限度地减少了内存复制。

```go
var b strings.Builder
b.WriteString("Hello")
b.WriteString(" ")
b.WriteString("World")
s := b.String() // s = "Hello World"
```

同时该包也提供两个方法，来控制缓冲区的大小。

- Grow()：用来预分配内存空间大小，如果实现知道所需内存大小，使用该函数来申请空间大小，这样减少后期的缓冲区大小操作，效率更高。
- Reset()：用于清空缓冲区。

## 二、性能分析

strings.Builder > bytes.Buffer  > []byte > fprift，直接相加

`strings.Builder`，`bytes.Buffer`，包括切片 `[]byte` 的内存是以倍数申请的。

### 1、strings.Builder和[]byte

strings.Builder省去了 `[]byte` 和字符串(string) 之间的转换，内存分配次数还减少了 1 次，内存消耗减半。

### 2、strings.Builder 和 bytes.Buffer

`strings.Builder` 和 `bytes.Buffer` 底层都是 `[]byte` 数组，但 `strings.Builder` 性能比 `bytes.Buffer` 略快约 10% 。一个比较重要的区别在于，`bytes.Buffer` 转化为字符串时重新申请了一块空间，存放生成的字符串变量，而 `strings.Builder` 直接将底层的 `[]byte` 转换成了字符串类型返回了回来。

## 三、相关wiki

- https://geektutu.com/post/hpg-string-concat.html