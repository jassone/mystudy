## type
todo
## 一、内置类型
比如int8，int16，byte，string，slice，map等。

## 二、自定义类型
使用type定义的类型，都是自定义类型。

## 三、类型定义和类型别名区别
```go
//类型定义
type NewInt int

//类型别名,
type MyInt = int
//还比如我们之前见过的rune和byte就是类型别名，他们的定义如下：
type byte = uint8
type rune = int32
```