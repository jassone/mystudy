## main

通过id()函数可以返回对象的整数标识

通过is可以判断是否为同一个对象

## 一、基础

### 1、定义

```
class Coffee:

  # __init__初始化
	def __init__(self,aaa=1,bbb,cccc=111):
		self.aaa=aaa
		...
```

### 2、继承

```
class Coffee(object):

调用父类：super.XXX()
```

### 3、类的多继承

C3算法，子类.\__mro\__ 打印继承关系

### 4、混入



