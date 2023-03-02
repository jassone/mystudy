## package

## 一、相关问题

### 1、包冲突

```go
import (
   "flag"
   "puzzlers/article3/q2/lib"
   lib2 "puzzlers/article3/q2/lib2/lib"
)

func main() {
   lib.Hello(name)
   lib2.Hello2(name)
}
```

### 2、将某个包公开

如果有代码包导入语句`import . fmt`，那么我们在当前源码文件中直接用`Printf`就可以了。在这个特殊情况下，程序在查找当前源码文件后会先去查用这种方式导入的那些代码包。

```go
. "fmt"
```

这样在当前包中就可以像在自己包中使用fmt包的程序实体。

##### 思考：公开代码包中的变量与当前代码包中的变量重名了，会怎么样？

- 全局的变量 ：会报redeclared。go会认为引入的代码包的代码，如同在本包中一样，那作用域其实是同一个，自然不允许重复声明。
- 函数体重新声明，作用域不一样，会出现屏蔽现象