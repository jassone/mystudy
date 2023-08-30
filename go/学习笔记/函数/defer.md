## defer

## 一、基础知识
### 1、defer特性：
* 关键字 defer 用于注册延迟调用。这些调用直到 return 前才被执。因此，可以用来做资源清理。

* 类似栈，**先进后出**
* 也可在panic 之前执行。

* **defer语句中的变量，在defer声明时就决定了**。

### 2、defer用途：
* 关闭文件句柄
* 锁的关闭
* 锁资源释放
* 数据库连接释放

### 2、defer执行流程：
1. 走到defer时，不会立即执行，而是将defer后的语句压到一个专门存储defer语句的栈中，然后继续执行函数下一个语句。

2. 当函数执行完后，再从defer栈中依次从栈顶中取出语句执行， 先进后出-逆序执行

3. 在defer语句放入栈中时，也会将相关的值复制进入栈中

### 3、相关demo

### 1、

```go
// 多个defer语句，按先进后出的方式执行。
func f11() {
   var whatever [5]struct{}
   for i := range whatever {
      defer fmt.Println(i) // 4,3,2,1,0 defer语句中的变量，在defer声明时就决定了。
   }
}

// 函数正常执行,由于闭包用到的变量 i 在执行的时候已经变成4,所以输出全都是4.
func main11() {
   var whatever [5]struct{}
   for i := range whatever {
      defer func() {  // 注意这里是闭包
        fmt.Println(i)   // 4 4 4 4 4
      }()
   }
}
```

### 2、优先计算函数参数

```go
func F(n int) func() int {
   return func() int {
      n++
      return n
   }
}
func main() {
   f := F(5)
   defer func() {
      fmt.Println(f()) //3  8
   }()
  defer fmt.Println(f()) //2 6  f()这里运到的时候就执行了。defer() 后面的函数如果带参数，会优先计算参数，并将结果存储在栈中，到真正执行 defer() 的时候取出。

   i := f()
   fmt.Println(i) //1 7
}
```

## 二、defer底层

### 1、1.12版本底层实现

defer信息会注册到一个链表，而当前执行的goroutine持有这个链表的头指针，每个goroutine在运行时都有一个对应的结构体g，其中一个字段指向defer链表头。

defer链表链起来的是一个一个_defer结构体，新注册的defer会添加到链表头，执行时也是从头开始，所以defer才会表现为倒序执行。

##### a) defer流程
![20220325192914.jpg](https://pic.imgdb.cn/item/623db41d27f86abb2a57bd21.jpg)

##### b) 编译后伪指令
![](https://pic.imgdb.cn/item/623db5b327f86abb2a60ee01.jpg)

##### c) defer结构体
![20220325193001.jpg](https://pic.imgdb.cn/item/623db43e27f86abb2a585b1a.jpg)

##### d) 缺点
![3-A682-69565D47DED8.png](https://pic.imgdb.cn/item/623db45727f86abb2a58969f.png)
* 从堆到栈，copy数据慢
* 链表慢

### 2、1.13底层优化(官方说提升30%)
##### a) 编译后伪指令
 ![20220325202708.jpg](https://pic.imgdb.cn/item/623db61f27f86abb2a64062b.jpg)

##### b) 优化点
把defer信息保持到当前函数栈帧局部变量里，再通过deferprocStack把defer信息注册到defer链接中。

优点是减少defer信息的堆分配，只需从栈上的局部变量空间拷贝到参数空间。

但是如果是循环中的defer，还是要用1.12版本中的方式，还是要有堆分配。

##### c) defer结构体
增加heap字段，标识是否是堆分配
 ![20220325204036.jpg](https://pic.imgdb.cn/item/623db85027f86abb2a738c7b.jpg)

### 3、1.14底层优化(几乎提升一个数量级)
该方式被称为：open coded defer
##### a) 编译后伪指令
 ![20220325204903.jpg](https://pic.imgdb.cn/item/623dba9327f86abb2a83c88d.jpg)

##### b) 优化点
* 不需要创建defer结构体
* 也不需要注册defer链表

但是依然不适用循环中的defer，所以1.12版本处理方式还保留。

##### c) 缺点
在伪指令code to do something地方如果有panic,runtime.Goexit，则后面的代码就执行不到，就要去执行defer链表了。

但是此时并没有defer链表，就需要额外通过栈扫码的方式来发现(通过defer结构体上某些信息找到未注册到链表的defer函数)。所以如果有panic，则会变慢。

##### c) defer结构体
在上一版本基础上增加几个字段
![20220325210705.jpg](https://pic.imgdb.cn/item/623dbe9a27f86abb2aa2bcba.jpg)
