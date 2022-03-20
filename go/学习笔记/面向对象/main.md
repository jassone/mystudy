## 面向对象
### 1、封装
##### 封装实现
* 将结构体、字段的首字母小写
* 给结构体所在包提供一个工厂模式的函数，首字母大写
* 提供一个首字母大写的Set方法，用于对属性判断并赋值
* 提供一个首字母大写的Get方法，用于获取属性的值
 
### 2、继承
如果一个结构体struct嵌套了另一个匿名结构体，那么这个结构体可以直接使用匿名结构体的字段和方法，从而体现继承。

##### 多重继承
如一个struct嵌套了多个匿名结构体，那么该结构体可以直接访问嵌套匿名结构体的字段和方法，从而实现多重继承

### 3、多态
接口体现多态的两种形式
##### a) 多态参数
即多个类型实现同一个接口。
##### b) 多态数组
```go
type Usb interface {
	Start()
}

// 手机结构体
type Phone struct {
	name string
}
// 让Phone 实现 Usb接口的方法
func (p Phone) Start() {
	fmt.Println("手机开始工作。。。")
}

// 相机结构体
type Camera struct {
	name string
}
// 让 Camera 实现 Usb 接口的方法
func (c Camera) Start() {
	fmt.Println("相机开始工作。。。")
}

func main() {
	// 这里就体现出多态数组
	var usbArr [3]Usb
	usbArr[0] = Phone{"vivo"}
	usbArr[1] = Phone{"小米"}
	usbArr[2] = Camera{"尼康"}
	fmt.Println(usbArr)
}
```
