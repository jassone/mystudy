## main

## 一、基础

- [12,23,34,"ererere"]表示
- 不可修改。
- **给一个函数作为参数传递时，函数内修改会影响到外部的列表**

## 二、常用操作

### 1、创建列表

- l = [1,2]
- list("red") - > ["r","e","d"]
- 列表推导式： [x for x in range(1,5)] -> [1,2,3,4]

### 2、访问列表

```
list[i]
list[i][j]
list[-1] // 最后一个
```

### 3、删除

```
del list[i]
del lsit // 删除list
```

### 4、添加元素

```
list.insert(索引, 元素)
list.append(元素) // 在列表结尾添加原始

list.extend(可迭代对象) 为列表扩展元素
list_demo("last") -> ["x","y","l","a","s","t"]
```

### 5、修改

```
list.remove(元素)
list.reverse() // 反转
list.pop(索引) // 移除索引位置对应的元素并返回该元素，不指定则移除最后一个
list.copy() // 复制列表
list.clear() // 清空列表
```

### 6、计算长度

```
len(list)
list.count(元素) // 某个元素出现的次数，注意是相等的判断看，不是包含的判断
```

### 7、排序

```
list.sort(reverse=true) // 原地排序
sort(list) // 排序后返回新的列表

```



