## slice
## 一、基础知识
切片是 Go 中的一种基本的数据结构，使用这种结构可以用来管理数据集合。**切片的设计想法是由动态数组概念而来**，为了开发者可以更加方便的使一个数据结构可以自动增加和减少。

**但是切片本身并不是动态数据或者数组指针。**它通过内部指针和相关属性引用数组片段，设定相关属性将数据读写操作**限定在指定的区域内**，以实现变长方案。切片本身是一个**只读对象**，其工作机制**类似数组指针的一种封装**。

终止索引标识的项不包括在切片内。切片提供了一个与**指向数组的动态窗口**。

### 1、切片的定义
var 变量名 []类型，
比如 var str []string  

### 2、切片内部结构
```go 
type slice struct { 
    array unsafe.Pointer  // 内存地址，即指向切片的数组的指针, 注意这里是指针
    len  int
    cap  int
}
var str []string  
// 这时候存放的内存地址为nil，len=0,cap=0
```
![reereedfd.jpeg](https://pic.imgdb.cn/item/6230b25b5baa1a80ab82ab03.jpg)

### 3、创建切片的各种方式
 见slice学习代码
 
### 4、切片初始化

```go
全局：
var arr = [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
var slice0 []int = arr[start:end] 
var slice1 []int = arr[:end]        
var slice2 []int = arr[start:]        
var slice3 []int = arr[:] 
var slice4 = arr[:len(arr)-1]      //去掉切片的最后一个元素
局部：
arr2 := [...]int{9, 8, 7, 6, 5, 4, 3, 2, 1, 0}
slice5 := arr[start:end]
slice6 := arr[:end]        
slice7 := arr[start:]     
slice8 := arr[:]  
slice9 := arr[:len(arr)-1] //去掉切片的最后一个元素
```

### 5、相关操作
![dfrtrfgf.jpeg](https://pic.imgdb.cn/item/6230b1d55baa1a80ab8274d7.jpg)

## 二、特性属性
### 1、切片的内存布局
![](https://pic.imgdb.cn/item/6230b29a5baa1a80ab82c271.jpg)
读写操作实际目标是底层数组，只需注意索引号的差别。

这里对y里的两个原始进行修改，x也会一起发生变化。

## 二、特性
* 切片（slice）是对数组一个连续片段的引用，所以切片是**引用类型**(占用资源很小)。**但自身是结构体，值拷贝传递。**
* 切片是对数组的一个连续‘片段’的引用
* 切片的长度可以改变，因此，切片是一个可变的数组。
* 切片遍历方式和数组一样，可以用len()求长度。表示可用元素数量，**读写操作不能超过该限制。** 
* mcap可以求出slice最大扩张容量，不能超出数组限制。0 <= len(slice) <= len(array)，其中array是slice引用的数组。
* 切片只能和nil比较，因为是引用类型，值类型可以比较(比如数组)
* 如果 slice == nil，那么 len、cap 结果都等于 0。
* slice 只能向后扩展，不能向前扩展

### 1、优化 todo
* 在大批量添加数据时，建议一次性分配足够大的空间，以减少内存分配和数据复制开销。或初始化足够长的 len 属性，改用索引号进行操作。及时释放不再使用的 slice 对象，避免持有过期数组，造成 GC 无法回收。

* 应及时将所需数据 copy 到较小的 slice，以便释放超大号底层数组内存。

## 三、底层原理
### 1、创建切片
make 函数允许在运行期动态指定数组长度，绕开了数组类型必须使用编译期常量的限制。

创建切片有两种形式
* make 创建切片
* 空切片

a) 下图是用 make 函数创建的一个 len = 4， cap = 6 的切片。内存空间申请了6个 int 类型的内存大小。由于 len = 4，所以后面2个暂时访问不到，但是容量还是在的。这时候数组里面每个变量都是0 。
![3434343.png](https://pic.imgdb.cn/item/6231b0a05baa1a80abf8582f.png)

b）字面量也可以创建切片。下面是用字面量创建的一个 len = 6，cap = 6 的切片，这时候数组里面每个元素的值都初始化完成了。需要注意的是 [ ] 里面不要写数组的容量，因为如果写了个数以后就是数组了，而不是切片了。
![76543.png](https://pic.imgdb.cn/item/6231b0e45baa1a80abf89072.png)

c）还有一种简单的字面量创建切片的方法。如下图，就 Slice A 创建出了一个 len = 3，cap = 3 的切片。从原数组的第二位元素(0是第一位)开始切，一直切到第四位为止(不包括第五位)。同理，Slice B 创建出了一个 len = 2，cap = 4 的切片。
![9876543.png](https://pic.imgdb.cn/item/6231b2d35baa1a80abf9f1a6.png)

d) **空切片**
![slice-8.png](https://pic.imgdb.cn/item/6231b3cd5baa1a80abfaae34.png)
	
空切片不能直接赋值
```go
	slice := make([]int,0)
	slice[0] = 1
```
### 2、nil 和空切片
a）**nil 切片**被用在很多标准库和内置函数中，描述一个不存在的切片的时候，就需要用到 nil 切片。比如函数在发生异常的时候，返回的切片就是 nil 切片。nil 切片的指针指向 nil。
![dfghjk.png](https://pic.imgdb.cn/item/6231b2f05baa1a80abfa0548.png)

对于nil，len和cap函数都将返回0。

b）空切片一般会用来表示一个空的集合。比如数据库查询，一条结果也没有查到，那么就可以返回一个空切片。

空切片和 nil 切片的区别在于，空切片指向的地址不是nil，指向的是一个内存地址，但是它没有分配任何内存空间，即底层元素包含0个元素。

不管是使用 nil 切片还是空切片，对其调用内置函数 append，len 和 cap 的效果都是一样的。

### 3、切片扩容策略
* 如果切片的容量小于 1024 个元素，于是扩容的时候就翻倍增加容量。

* 一旦元素个数超过 1024 个元素，那么增长因子就变成1.25 ，即每次增加原来容量的四分之一。

注意：扩容扩大的容量都是**针对原来的容量**而言的，而不是针对原来数组的长度而言的。

### 4、切片扩容后是新的还是旧的
a) 未超过cap
![slice-10.png](https://pic.imgdb.cn/item/6231c3d95baa1a80ab095924.png)

由于原数组还有容量可以扩容，所以执行 append() 操作以后，并没有新建一个新的数组，扩容前后的数组都是同一个，这也就导致了新的切片修改了一个值，也影响到了老的切片了。

并且 append() 操作也改变了原来数组里面的值。一个 append() 操作影响了这么多地方，如果原数组上有多个切片，那么这些切片都会被影响！无意间就产生了莫名的 bug！

这种情况也极容易出现在字面量创建切片时候，第三个参数 cap 传值的时候，如果用字面量创建切片，cap 并不等于指向数组的总容量，那么这种情况就会发生。

``` go
slice := array[1:2:3]
```
上面这种情况非常危险，极度容易产生 bug 。

建议用字面量创建切片的时候，cap 的值一定要保持清醒，避免共享原数组导致的 bug。

// 比如下面这个例子
```go
func test(arr []int) []int {
	fmt.Printf("res2: %p\n", &arr)
	arr[0] = 2
	fmt.Printf("res3: %p\n", &arr)
	return arr
}
func main() {
	var arr []int
	arr = []int{1}
	fmt.Printf("res1: %p\n", &arr)

	fmt.Println(test(arr))
	fmt.Printf("res4: %p\n", &arr)
	fmt.Println((arr))
}
```
res1和res4一样，res2和res3一样，但是res2和res1不一样。
因为切片是值拷贝(其实这里说是引用拷贝会更好点)，所以会新开个切片，但是初始时底层数组是一样的，所以修改底层数组时，会影响到原来的。看起来像是引用传递

b) 超过cap
![slice-9.png](https://pic.imgdb.cn/item/6231c3b85baa1a80ab091b35.png)

因为原来数组的容量已经达到了最大值，再想扩容， Go 默认会先开一片内存区域，把原来的值拷贝过来，然后再执行 append() 操作。这种情况丝毫不影响原数组。

新切片指向的数组是一个全新的数组。并且 cap 容量也发生了变化。

> 建议尽量避免情况一，尽量使用情况二，避免 bug 产生。

// 比如下面这个例子
```go
 func test(arr []int) []int {
   arr[0] = 999 //但是这行会影响到原来的
	fmt.Printf("res2: %p\n", &arr)
	arr = append(arr, 888)
	fmt.Printf("res3: %p\n", &arr)
	return arr
}
func main() {
	var arr []int
	arr = []int{1}
	fmt.Printf("res1: %p\n", &arr)

	fmt.Println(test(arr))
	fmt.Printf("res4: %p\n", &arr)
	fmt.Println((arr))
}
```
同上，res1和res4一样，res2和res3一样，但是res2和res1不一样。
修改底层数组时，因为会扩容，底层数组会新开，会影响到原来的。所以不会修改到原来的数组。

### 5、切片拷贝
copy 方法会把源切片值中的元素复制到目标切片中，**并返回被复制的元素个数**，copy 的两个类型必须一致。最终的复制结果取决于较短的那个切片，当较短的切片复制完成，整个复制过程就全部完成了。

![slice-11.png](https://pic.imgdb.cn/item/6231cc2b5baa1a80ab18a4af.png)

### 6、append
**对于空切片，nil切片，此时如果用append添加元素，则可以分配底层数据，可以正常操作。**

一个扩容后需要空间大小估算
![5AEF610B-9780-4610-B10D-122EFE608A0D.png](https://pic.imgdb.cn/item/623d69ff27f86abb2aa7a995.png)

### 7、切片value的值拷贝

```
func main() {
    slice := []int{10, 20}
    for index, value := range slice {
        fmt.Printf("value = %d , value-addr = %x , slice-addr = %x\n", value, &value, &slice[index])
    }
}
输出
value = 10 , value-addr = c4200aedf8 , slice-addr = c4200b0320
value = 20 , value-addr = c4200aedf8 , slice-addr = c4200b0328
```
![slice-12.png](https://pic.imgdb.cn/item/6231cd0e5baa1a80ab1a8988.png)

如果用 range 的方式去遍历一个切片，拿到的 Value 其实是**切片里面的值拷贝**。所以每次打印 Value 的地址都不变。

由于 Value 是值拷贝的，并非引用传递，所以直接改 Value 是达不到更改原切片值的目的的，需要通过 &slice[index] 获取真实的地址。

## 四、相关wiki
* https://www.jianshu.com/p/030aba2bff41
