## 函数
**函数的类型只和参数和返回值相关。**

## 一、基础知识
### 1、基本特性
* 函数是一等公民：参数，变量，返回值都可以是函数。
* 可变参数本质上就是 slice。只能有一个，且必须是最后一个。

### 2、函数声明：
函数声明包含
* **函数名**
* **参数列表**
* **返回值列表**
* **函数体**

```go
func funca() (n int) {
   n = 12 // 这里不能使用 n:=12,因为返回值那里已经声明了
   return // 这里可以省略n
}
```

### 3、匿名函数
匿名函数是一种没有函数名的函数，即定义即使用。

匿名函数没有函数名，只有函数体，它只有在被调用的时候才会初始化。匿名函数一般被当作一种类型赋值给函数类型的变量，经常被用作`回调函数`。

## 二、函数转换

### 1、注意这里handler(doubler)不是调用，而是在进行函数转换

```go
//定义函数类型
type handler func(name string) int

//另一个函数定义
func doubler(name string) int {
	return len(name) * 2
}

// ******doubler函数显式转换成handler函数对象******
fmt.Println(handler(doubler))
```

## 三、函数返回

### 1、返回demo

return步骤：**1 给返回值赋值，2 处理defer,  3 返回返回值**。

```go
func fr1() int {
   x := 5
   defer func() {
      x++ // 修改的是x，不影响返回值
   }()
   return x // 返回5，返回值未命名，所以返回值就是这里的x=5
}

func fr2() (x int) { // x初始化0， 并且作用域为该函数全域
	x = 5 // x :=5 错误
	defer func() {
		x++ // 返回值x++，
	}()
	return 5 // 返回6， 这里设置返回值5，
}

// 先计算返回值为x=6= 5+1，再处理defer x=7, 再给返回值赋值7
func fr22() (x int) {
	x = 5
	defer func() {
		x++ // 返回值x++
	}()
	return x + 1 // 返回7， 这里设置返回值x=5+1=6，
}

func fr3() (y int) {
	x := 5
	defer func() {
		x++ // 这里不影响y
	}()
	return x // 返回5, y=x=5  第一步先给返回值赋值
}

func fr33() (z int) {
	x, y := 1, 2
	defer func() {
		z += 100
	}()

	z = x + y
	return // 返回103
}

func fr4() (x int) {
	x = 5
	defer func(x int) {
		x++ // 这里x是func内部的x变量，作用域仅在func内
	}(x)
	return 5 // 返回5
}

func fr5() (x int) {
	x = 5
	defer func(x int) int {
		x++      // 这里x是func内部的x变量，作用域仅在func内
		return x //虽然这里返回，没人接受，所以不影响外部x的值
	}(x)
	return 5 // 返回5
}

func fr6() (x int) {
	x = 5
	defer func(x *int) {
		(*x)++ // 这里指针操作，影响外部
	}(&x)
	return 5 // 返回6
}
```

### 2、“裸”返回

```go
/*
如果return了具体值，则会赋值给返回变量
如果return后面为空，则返回各个返回变量的当前值。这种用法被称作“裸”返回。
*/
func add1(a, b int) (c int) { // 返回值不能定义为 a int, 报duplicate argument a
   c = a + b
   return
}
```
