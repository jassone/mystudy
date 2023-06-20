## range

## 一、iteration variable(迭代变量)重用
用for range的hort variable declaration（:=）形式在for expression中声明iteration variable时，需要注意的是这些variable在每次循环体中都会被重用，而不是重新声明。

```go
var m = [...]int{1, 2, 3, 4, 5}

for i, v := range m {
    go func() {
        fmt.Println(i, v)
    }()
}
// 输出5个    4 5
```
各个goroutine中输出的i,v值都是for range循环结束后的i, v最终值，而不是各个goroutine启动时的i, v值。

修正一下
```go
for i, v := range m {
    go func(i, v int) {
        fmt.Println(i, v)
    }(i, v)
}
```

## 二、range expression副本参与iteration
range后面接受的表达式的类型包括：**array, pointer to array, slice, string, map和channel(有读权限的)**。

我们以array为例来看一个简单的例子：
```go
var a = [5]int{1, 2, 3, 4, 5}
var r [5]int
for i, v := range a { //相当于复制了个a'
    if i == 0 {
        a[1] = 12
        a[2] = 13
    }
    r[i] = v
}
fmt.Println("r = ", r)
fmt.Println("a = ", a)
// 输出
//r =  [1 2 3 4 5]
//a =  [1 12 13 4 5]
```
可以看出range expression副本参与循环。也就是说在上面这个例子里，真正参与循环的是a的副本，而不是真正的a。

**Go中的数组在内部表示为连续的字节序列，虽然长度是Go数组类型的一部分，但长度并不包含在数组的内部表示中，而是由编译器在编译期计算出来。**这个例子中，对range表达式的拷贝，即对一个数组的拷贝，新拷贝的a'则是Go临时分配的连续字节序列，与a完全不是一块内存。因此无论a被 如何修改，其副本a'依旧保持原值，并且参与循环的是a'，因此v从a'中取出的仍旧是a的原值，而非修改后的值。

## 三、看下pointer to array的情况

```go
var a = [5]int{1, 2, 3, 4, 5}
var r [5]int
for i, v := range &a {
    if i == 0 {
        a[1] = 12
        a[2] = 13
    }
    r[i] = v
}
fmt.Println("r = ", r)
fmt.Println("a = ", a)
// 打印都是  [1 12 13 4 5]
```

我们看到这次r数组的值与最终a被修改后的值一致了。这个例子中我们使用了\*[5]int作为range表达式，**其副本依旧是一个指向原数组 a的指针**，因此后续所有循环中均是&a指向的原数组亲自参与的，因此v能从&a指向的原数组中取出a修改后的值。

## 四、看下slice的情况
diomatic go建议我们尽可能的用slice替换掉array的使用。

```go
var a = [5]int{1, 2, 3, 4, 5}
var r [5]int

for i, v := range a[:] {
	if i == 0 {
		a[1] = 12
		a[2] = 13
	}
	r[i] = v
}
fmt.Println("r = ", r)
fmt.Println("a = ", a)
// 打印都是  [1 12 13 4 5]
```

显然用slice也能实现预期要求。我们可以分析一下slice是如何做到的。

slice在go的内部表示为一个struct，由(\*T, len, cap)组成，其中\*T指向slice对应的underlying array的指针，len是slice当前长度，cap为slice的最大容量。**当range进行expression复制时，它实际上复制的是一个 slice，也就是那个struct。副本struct中的\*T依旧指向原slice对应的array**，为此对slice的修改都反映到 underlying array a上去了，v从副本struct中*T指向的underlying array中获取数组元素，也就得到了被修改后的元素值。

slice与array还有一个不同点，就是其len在运行时可以被改变，而array的len是一个常量，不可改变。那么len变化的 slice对for range有何影响呢？我们继续看一个例子：

```go
var a = []int{1, 2, 3, 4, 5}
var r = make([]int, 0)
for i, v := range a {
	if i == 0 {
		a = append(a, 6, 7)
	}
	r = append(r, v)
}
fmt.Println("r = ", r)
fmt.Println("a = ", a)
// r =  [1 2 3 4 5]
// a =  [1 2 3 4 5 6 7]
```

在这个例子中，原slice a在for range过程中被附加了两个元素6和7，其len由5增加到7，但这对于r却没有产生影响。这里的原因就在于**a的副本a'的内部表示struct中的 len字段并没有改变**，依旧是5。

**range的副本行为会带来一些性能上的消耗，尤其是当range expression的类型为数组时，range需要复制整个数组；而当range expression类型为pointer to array或slice时，这个消耗将小得多。**

## 五、看下string的情况

对string来说，由于string的内部表示为struct {*byte, len)，并且string本身是immutable的，因此其行为和消耗和slice expression类似。不过for range对于string来说，每次循环的单位是rune(code point的值)，而不是byte，index为迭代字符码点的第一个字节的position：

```go
var s = "中国"
for i, v := range s {
    fmt.Printf("%d %s 0x%x\n", i, string(v), v)
}
// 0 中 0x4e2d
// 3 国 0x56fd
```

如果s中存在非法utf8字节序列，那么v将返回0xFFFD这个特殊值，并且在接下来一轮循环中，v将仅前进一个字节：
```go
var sl = []byte{0xe4, 0xb8, 0xad, 0xe5, 0x9b, 0xbd, 0xe4, 0xba, 0xba}
for _, v := range sl {
	fmt.Printf("0x%x ", v)
}
fmt.Println("\n")

sl[3] = 0xd0
sl[4] = 0xd6
sl[5] = 0xb9

for i, v := range string(sl) {
	fmt.Printf("%d %x\n", i, v)
}
输出
0 4e2d
3 fffd
4 5b9
6 4eba
```

## 六、看下map的情况
对于map来说，map内部表示为一个指针，指针副本也指向真实map，因此for range操作均操作的是源map。

for range不保证每次迭代的元素次序，对于下面代码：

```go
var m = map[string]int{
	"tony": 21,
	"tom":  22,
	"jim":  23,
}

for k, v := range m {
	fmt.Println(k, v)
}
```

如果map中的某项在循环到达前被在循环体中删除了，那么它将不会被iteration variable获取到。

```go
var m = map[string]int{
	"tony": 21,
	"tom":  22,
	"jim":  23,
}
counter := 0
for k, v := range m {
	if counter == 0 {
		delete(m, "tony")
	}
	counter++
	fmt.Println(k, v)
}
fmt.Println("counter is ", counter)
```
同理：如果在循环体中新创建一个map元素项，那该项元素可能出现在后续循环中，也可能不出现。

## 七、看看channel的情况
对于channel来说，channel内部表示为一个指针，channel的指针副本也指向真实channel。

for range最终以阻塞读的方式阻塞在channel expression上（即便是buffered channel，当channel中无数据时，for range也会阻塞在channel上），直到channel关闭：

```go
var c = make(chan int)
go func() {
	time.Sleep(time.Second * 3)
	c <- 1
	c <- 2
	c <- 3
	close(c)
}()
for v := range c {
	fmt.Println(v)
}
```

如果channel变量为nil，则for range将永远阻塞。

## 八、注意点
### 1、range中获取k,v的区别

```go
// 1、range中:=左边一个key的情况，为获取的下标
x := map[string]string{"a":"b"}
for v:= range x {
	fmt.Print(v) // a
}

x := []string{"a", "b", "c"}
for v:= range x {
	fmt.Print(v) // 0 1 2
}

x := [3]int{2,3,4}
for v:= range x {
	fmt.Print(v) // 0 1 2
}

// 2、通道中的情况
var c = make(chan int)
for v := range c { // 获取值,未关闭则一直阻塞，如果关闭了，则退出循环
	fmt.Println(v)
}
```

## 九、range语法

```
// 不取值
for range p {
}
// 取一个
for v := range p {
}
// 取二个
for k,v := range p {
}
```

## 十、总结
range 中改变会影响源变量的情况有
* pointer to array
* slice
* map
* channel

