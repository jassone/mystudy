## json使用的一些技巧

## 一、基础知识

### 1、go将字符串转换为json对象时，是按照这样映射转换的

```
boolean >> bool
number  >> float64 // 特别要注意不是int
string  >> string
null    >> nil
array   >> []interface{}
object  >> map[string]interface{} // 注意value是interface类型
```



### 2、其他

```go
type Person struct {
    Name string
    Age  int `json:",omitempty"`
}
// omitempty的使用  https://www.jianshu.com/p/b826e7f297ea
```



```go
// playload不一定是map或者struct,其他简单类型也可以
resp,_ := json.Marshal(playload)
```

https://www.liwenzhou.com/posts/Go/json-tricks/

https://www.jianshu.com/p/922c5a6f51e5

## 二、相关问题

### 1、精度丢失问题

 todo https://www.51cto.com/article/697019.html

## 相关优化

- 将标准的json库替换为jsonitor，大概能提高数倍序列化的能力。
- 还有字节的soni库性能也不错

