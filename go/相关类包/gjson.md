## gjson

## 一、基础知识

### 1、go将字符串转换为json对象时，是按照这样映射转换的

```csharp
boolean >> bool
number  >> float64 // 特别要注意不是int
string  >> string
null    >> nil
array   >> []interface{}
object  >> map[string]interface{} // 注意value是interface类型
```

## 相关wiki
* golang json库gjson的使用 https://www.jianshu.com/p/623f8ca5ec12