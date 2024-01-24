## main

## 一、基础

整数1和1.0是相同的键



### 1、定义

{"one":1}



## 二、常用操作

### 1、创建

```
{}直接定义
使用类型构造器： dist(one=1,two=2)
使用字典推导式：{x: x**2 for x in range(10)}
```

### 2、删除

```
del 删除字典
del m["aaa"] 删除指定元素
```

### 3、访问

```\
m
m.items()
m.get("aaa")
m["aaa"]

m.keys()
m.values()

for key,value in m.items():
	print(key)
	print(value)
```

### 4、修改

```
m["aaa"] = 1213
```

### 5、其他

```
len()
key in m // key是否在m中
pop() // 如果键在字典中，移除该键，返回该键对应的值
popitem() // 如果键在字典中，移除该键，返回键值对

m.setdefault(key[,default]) // 如果字典存在键key，返回对应的值。否则插入default的键key，并返回default
如果没有default，值为None

m |= new_m // 用新字典的键值更新旧的字典 （>=3.9）

dict(zip(列表1, 列表2)) // 合并两个列表为字典

```

## 三、其他

### 1、相关报错

KeyError:"XXX"  // 字典中没有该key

解决：先判断，后操作， 比如  aa in m
