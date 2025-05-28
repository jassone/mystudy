## main

## 一、基础

### 特点

- 定义的形参，**不用指定数据类型**，会根据传入的实参决定

  

### 1、匿名函数 - lambda

```
add_1 = lambda x:x+1   // 只能是一行代码
print(add_1(10))
```

### 2、不定长函数

```
接收参数是一个元祖
def sum(*args):
	for ele in args:
	  total += ele

关键字可变参数,接收关键字参数字典格式
def address(**args):
	for arg_name in args:
	  print(args[arg_name])
address(name="jack",age=20)

def address2(name,*telphone,alias_name=None,**custom)
* 接收位置参数，格式是元祖
** 接收关键字参数，未定义的参数会给到custom ???

```



### 3. 关键字传参 - 不受参数位置影响

```
def book(name, age):
    print(name, age)

book(age=12, name="jackl")
book(12, name="jackl")
```

### 4、默认参数

- 默认参数需要放在最后

```
def book(addr,name="jack", age=12):
    print(name, age)

book("shanghai")
book("shanghai",age=13)
```

### 5、参数为函数

```
def max_number(a, b):
    if a > b:
        return a
    else:
        return b

def get_max(fun, a, b):
    return fun(a, b)

print(get_max(max_number, 5, 3))
```



### 其他

```
dir(funcname) // 打印一些魔术方法
有一个__call__方法，所以funcname()可以执行
```



## 二、高阶函数

### 1、map

内置函数，直接使用。

map(函数，可迭代对象)

- 函数：函数对象、lambda表达式
- 可迭代对象：列表、元祖、range等
- 将可迭代对象的每个元素作为函数参数执行

```
def add(number):
	return (number+number)
for i in map(add,range(5))
	print(i)
	
简化为：list(map(lambda x:x+x,range(5)))
```

 

### 2、filter

内置函数，直接使用.

filter(函数，可迭代对象)

- 过滤可迭代对象中返回值不为True的元素
- filter函数返回一个可迭代对象

```
for i in filter(lambda x:x>0,(-1,-2,1,2))
	print(i)
```

### 3、reduce

reduce(函数，可迭代对象)

- 可迭代对象进行累积

```
form functools import reduce
reduce(lambda x ,y:x+y,[1,2,3])
结果：1+2+3
```

## 三、偏函数

 ## 四、装饰器

```
def name(n):
    def func(x):
        res = n(x+x)
        return res
    return func
 
@name
def run(x):
    print(x)

run(1)
```

- 自带内置装
- 其他饰器：functools包

- 常用的：lrucache 缓存，wraps 被装饰函数保持原对象不变（普通的装饰器会改变函数名）



## 注意

- 同一个文件，出现两个函数名相同的函数，则就近调用？？？
- 
