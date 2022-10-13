## 汇总



### 可变参数

可变参数可以保证无法传入`null`，因为传入0个参数时，接收到的实际值是一个空数组而不是`null`。



### 任何`class`都有构造方法

如果一个类没有定义构造方法，编译器会自动为我们生成一个默认构造方法，它没有参数，也没有执行语句，类似这样：

```
class Person {
    public Person() {
    }
}
```

### 成员变量初始值

```java
class Person {
    private String name; // 默认初始化为null,注意不是空字符串。
    private int age; // 默认初始化为0
    private bool flag; // 默认初始化为false

    public Person() {
    }
}
```



### 公共类

public class XXX {}, 公共类的类名和文件名必须相同。



### 其他特点

- 一个java程序中类名相同的类只能有一个，也就是类型不能重名。
- Java支持嵌套类，如果一个类内部还定义了嵌套类，那么，嵌套类拥有访问`private`的权限。

### final

可作用于class, method, field,局部变量(可以阻止被重新赋值)

```java
protected void hi(final int t) {
  t = 1; // error!
}
```



### 其他特性

类就是一种自定义类型。 