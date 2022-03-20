## http
todo

```go
res, err := http.Get("http://xxxxxxxxxx")
if res != nil {
    defer res.Body.Close()
}

if err != nil {
    return err
}
```
需要检查 res 的值是否为 nil ，这是 http.Get 中的一个警告。通常情况下，出错的时候，返回的内容应为空并且错误会被返回，可当你获得的是一个重定向 error 时， res 的值并不会为 nil ，但其又会将错误返回。上面的代码保证了无论如何 Body 都会被关闭，如果你没有打算使用其中的数据，那么你还需要丢弃已经接收的数据。