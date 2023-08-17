## json标签

### 1、omitempty

```go
type Person struct {
    Name string
    Age  int `json:",omitempty"`
}
// omitempty的使用，如果为空，则忽略Age  https://www.jianshu.com/p/b826e7f297ea
```

