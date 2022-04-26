## map
map是一种**无序**的基于key-value的数据结构，**map是引用类型，必须初始化才能使用。**

## 一、基础知识
### 1、语法
map的定义语法如下
```go
    map[KeyType]ValueType
```
* KeyType:表示键的类型。

* ValueType:表示键对应的值的类型。

**map类型的变量默认初始值为nil，需要使用make()函数来分配内存**。
语法为：
```go
make(map[KeyType]ValueType, [cap])
```
其中cap表示map的容量，该参数虽然不是必须的，但是我们应该在初始化map的时候就为其指定一个合适的容量。

### 2、基本使用
map中的数据都是成对出现的。
```go
scoreMap := make(map[string]int, 8)
scoreMap["张三"] = 90
```

### 3、判断某个键是否存在
语法
```go
value, ok := map[key]
if ok {
    // 存在
} 
```
这种方式叫做：ok-idiom（A跌目）模式：多返回值中用一个名为ok的布尔值来标记操作是否成功。

**value返回零值。**

### 4、map的遍历
使用range

遍历map时的元素顺序与添加键值对的顺序无关。

### 5、使用delete()函数删除键值对
```go
delete(map, key)
```

### 6、特性
* 字典并不是并发安全
* **map 只能和 nil对比, 所以不能 m1 == m2 这样比较。**
* map是hash，无序的
* map不能用new()来构建，因为会分配一个引用对象，获得一个空引用指针，相当于声明了一个未初始化的变量并取他的地址。
* **获取一个不存在的key值，不会报错，会返回零值。**

## 二、map值跟下
### 1、map 中的元素是不可寻址的
比如 map 一个字段的值是 struct 类型，则无法直接更新该 struct 的单个字段。
```go
type data struct {
    name string
}

func main() {
    m := map[string]data{
        "x": {"Tom"},
    }
    m["x"].name = "Jerry"// panic
}

// slice可以
func main2() {
    s := []data{{"Tom"}}
    s[0].name = "Jerry"
    fmt.Println(s)    // [{Jerry}]
}
```

### 2、更新 map 中 struct 元素的字段值，有 2 个方法

##### a) 使用局部变量
提取整个 struct 到局部变量中，修改字段值后再整个赋值

```go
type data struct {
    name string
}
func main() {
    m := map[string]data{
        "x": {"Tom"},
    }
    r := m["x"]
    r.name = "Jerry"
    m["x"] = r
    fmt.Println(m)    // map[x:{Jerry}]
}
```
##### b) 使用指向元素的 map 指针

```go
func main() {
    m := map[string]*data{
        "x": {"Tom"},
    }

    m["x"].name = "Jerry"    // 直接修改 m["x"] 中的字段
    fmt.Println(m["x"])    // &{Jerry}
}
```

但是要注意下边这种误用：
```go
func main() {
    m := map[string]*data{
        "x": {"Tom"},
    }
    m["z"].name = "what???"     
    fmt.Println(m["x"])
}
```