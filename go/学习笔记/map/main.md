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