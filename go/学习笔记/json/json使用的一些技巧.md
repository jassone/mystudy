## json使用的一些技巧

todo



```go
// playload不一定是map或者struct,其他简单类型也可以
resp,_ := json.Marshal(playload)
```

https://www.liwenzhou.com/posts/Go/json-tricks/

https://www.jianshu.com/p/922c5a6f51e5



## 相关优化

- 将标准的json库替换为jsonitor，大概能提高数倍序列化的能力。

