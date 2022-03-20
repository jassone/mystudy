## sync.Once
Go语言中的sync包中提供了一个针对只执行一次场景的解决方案–sync.Once。

sync.Once只有一个Do方法，其签名如下：
```go
func (o *Once) Do(f func()) {}
```

注意：如果要执行的函数f需要传递参数就需要搭配闭包来使用。

```go
	var once sync.Once
	onceBody := func() {
		fmt.Println("Only once")
	}
	done := make(chan bool)

	for i := 0; i < 10; i++ {
		go func() {
			once.Do(onceBody)
			done <- true
		}()
	}
	for i := 0; i < 10; i++ {
		<-done
	}
```
sync.Once其实内部包含一个互斥锁和一个布尔值，互斥锁保证布尔值和数据的安全，而布尔值用来记录初始化是否完成。这样设计就能保证初始化操作的时候是并发安全的并且初始化操作也不会被执行多次。

