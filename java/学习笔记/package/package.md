## package

Java定义了一种名字空间，称之为包：`package`。一个类总是属于某个包，类名（比如`Person`）只是一个简写，真正的完整类名是`包名.类名`。

## 包作用域

位于同一个包的类，可以访问包作用域的字段和方法。



## import

如果要引入其他包，有三种方法：

### 1、直接写出完整类名

很麻烦

```java
mr.jun.Arrays arrays = new mr.jun.Arrays();
```

### 2、先import类名目录，再使用

```java
// 导入完整类名:
import mr.jun.Arrays;
// import mr.jun.*; //  导入mr.jun包的所有class（但不包括子包的class）
// 一般不推荐，因为导入了多个包后，不知道Arrays类属于哪个包
public class Person {
    public void run() {
        Arrays arrays = new Arrays();
    }
}
```

### 3、import static

`import static`的语法，它可以导入可以导入一个类的静态字段和静态方法。很少用。

```java
// 导入System类的所有静态字段和静态方法:
import static java.lang.System.*;
public class Main {
    public static void main(String[] args) {
        // 相当于调用System.out.println(…)
        out.println("Hello, world!");
    }
}
```



## 编译

Java编译器最终编译出的`.class`文件只使用*完整类名*，因此，在代码中，当编译器遇到一个`class`名称时：

- 如果是完整类名，就直接根据完整类名查找这个`class`；
- 如果是简单类名，按下面的顺序依次查找：
  - 查找当前`package`是否存在这个`class`；
  - 查找`import`的包是否包含这个`class`；
  - 查找`java.lang`包是否包含这个`class`。

如果按照上面的规则还无法确定类名，则编译报错。

```java
import java.text.Format;
public class Main {
    public static void main(String[] args) {
        java.util.List list; // ok，使用完整类名 -> java.util.List
        Format format = null; // ok，使用import的类 -> java.text.Format
        String s = "hi"; // ok，使用java.lang包的String -> java.lang.String
        System.out.println(s); // ok，使用java.lang包的System -> java.lang.System
        MessageFormat mf = null; // 编译错误：无法找到MessageFormat: MessageFormat cannot be resolved to a type
    }
}
```

因此，编写class的时候，编译器会自动帮我们做两个import动作：

- 默认自动`import`当前`package`的其他`class`；
- 默认自动`import java.lang.*`。

注意：

- 自动导入的是java.lang包，但类似java.lang.reflect这些包仍需要手动导入。
- 如果有两个`class`名称相同，例如，`mr.jun.Arrays`和`java.util.Arrays`，那么只能`import`其中一个，另一个必须写完整类名。

### 1、最佳实践

为了避免名字冲突，我们需要确定唯一的包名。推荐的做法是使用倒置的域名来确保唯一性。例如：

- org.apache
- org.apache.commons.log
- com.liaoxuefeng.sample

子包就可以根据功能自行命名。

要注意不要和`java.lang`包的类重名，即自己的类不要使用这些名字：

- String
- System
- Runtime

要注意也不要和JDK常用类重名：

- java.util.List
- java.text.Format
- java.math.BigInteger