## simplejson

golang标准库的json需要预先定义好结构体，然后才能将json字符串转化为golang的结构体；simplejson这个开源的库可以在不知道json字符串具体结构的情况下进行编码和解码

## 使用

```go
func case1() {
  //初始化*simpleJson.Json对象
  sj, err := simplejson.NewJson([]byte(jsonStr))

  var v *simpleJson.Json

  //获取字段，如果有多级，可以层层嵌套获取
  v = sj.Get(字段名1).Get(字段名2)

  //将v的值转化为具体类型的值，MustXXX方法一定可以转化成功
  //若转化不成功，则转化为该类型的零值
  result := v.MustString()
}

func case2() {
  //检查某个字段是否存在 
  _, ok := js.Get("字段名1").CheckGet("字段名2") 
  if ok { 
    fmt.Println("存在！") 
  } else { 
    fmt.Println("不存在") 
  }

}
```

todo

https://studygolang.com/articles/25789

https://pkg.go.dev/github.com/bitly/go-simplejson