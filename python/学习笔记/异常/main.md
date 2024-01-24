## main

## 一、基础

### 1、异常基类

- BaseException - 所有内置异常的基类
- Exception - 所有内置的非系统退出类异常都派生自此类，所有用户自定义异常也应当派生自此类
- ArithmeticError
- BufferError
- LookupError

### 2、捕获

```
try-except代码块

try:
except:
else: 不抛出异常，执行
finally:不管抛不抛异常，都执行
```



## 二、自定义异常

```
class MyException(Exception):
```

