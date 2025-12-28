## range
range关键字是Go语言中一个非常有用的迭代array，slice，map, string, channel中元素的内置关键字。

## 一、特性
### 1、迭代切片
range表达式在开始执行循环之前就已经构建了。
```go
func modifySlice() {
    v := []int{1, 2, 3, 4}
    for i := range v {
        v = append(v, i)
        fmt.Printf("Modify Slice: value:%v\n", v)
    }
}
输出
Modify Slice: value:[1 2 3 4 0]
Modify Slice: value:[1 2 3 4 0 1]
Modify Slice: value:[1 2 3 4 0 1 2]
Modify Slice: value:[1 2 3 4 0 1 2 3]
```
### 2、迭代map
```go
func modifyMap() {
    data := map[string]string{"a": "A", "b": "B", "c": "C"}
    for k, v := range data {
        data[v] = k
        fmt.Println("modify Mapping", data)
    }
}
```
**输出结果次数>=3**

##### 原因分析
对于**map的迭代来说，map中内容的迭代顺序没有指定，迭代map的时候将以随机的顺序返回里面的内容**。

* 如果一个map的key在还没有被迭代到的时候就被delete掉了，那么它所对应的迭代值就不会被产生了。
* 对于在迭代过程中新增加的key，则它可能会被迭代到也可能不会。

### 3、迭代字符串
使用range迭代字符串时，**range迭代的是Unicode而不是字节。**返回的两个值：
* 第一个是被迭代的字符的UTF-8编码的第一个字节在字符串中的索引。所以索引值可能并不是连续的。

* 第二个值为对应的字符且类型为rune(实际就是表示unicode值的整形数据）。不是string类型的，如果要使用该值需要格式化。

```go
//string
func rangeString() {
    datas := "aAbB"

    for k, d := range datas {
        fmt.Printf("k_addr:%p, k_value:%v\nd_addr:%p, d_value:%v\n----\n", &k, k, &d, d)
        fmt.Println(string(rune(d))) // 转换为字符
    }
}

输出
k_addr:0xc0000b2008, k_value:0 d_addr:0xc0000b2010, d_value:97
----
k_addr:0xc0000b2008, k_value:1 d_addr:0xc0000b2010, d_value:65
----
k_addr:0xc0000b2008, k_value:2 d_addr:0xc0000b2010, d_value:98
----
k_addr:0xc0000b2008, k_value:3 d_addr:0xc0000b2010, d_value:66
----
```

### 4、range表达式是指针
只有array类型能以指针的形式出现在range表达式中，其他都会报错。

```go
//compile error: cannot range over datas (type *string)
//d := "aAbBcCdD"
d := [5]int{1, 2, 3, 4, 5} //range successfully
//d := []int{1, 2, 3, 4, 5} //compile error: cannot range over datas (type *[]int)
//d := make(map[string]int) //compile error: cannot range over datas (type *map[string]int)
datas := &d

for k, d := range datas {
    fmt.Printf("k_addr:%p, k_value:%v\nd_addr:%p, d_value:%v\n----\n", &k, k, &d, d)
}

```

## 二、wiki 
* https://www.jianshu.com/p/4205659ca419