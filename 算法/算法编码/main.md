## 算法编码
## 一、一般算法
### 1、递归
三部曲
* 找出截止条件
* 找出下一个循环的返回值
* 当前的返回值

比如下面的等比计算
```go
func dengbi(start, num, step int) int {
	// 找出截止条件
	if num == 1  {
		return start
	}

	// 找出下一个循环的返回值
	next := dengbi(start,num - 1 ,step)

	// 当前的返回值(根据下一个循环的返回值求得)
	return  next * step
}
```

## 时间轮算法 todo