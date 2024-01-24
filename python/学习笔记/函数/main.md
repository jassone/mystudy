## main

## 一、基础

### 1、匿名函数

```
add_1 = lambda x:x+1
print(add_1(10))
```

### 2、不定长函数

```
def address(name,*telphone,alias_name=None,**custom)
* 接收位置参数，格式是元祖
** 接收关键字参数，未定义的参数会给到custom，字典格式

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

