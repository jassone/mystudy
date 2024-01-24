## main

## 一、基础

- 字符串不可修改



## 二、基础用法

### 1、f-string

Python f-string 是执行字符串格式化。

```python
name = "张三"
age = 18
# 最原始提供的字符串替换方法，使用了 % 运算符和经典字符串格式指定，如 %s %d 等
print("【%s】今年【%d】岁" % (name, age))  # 【张三】今年【18】岁
# Python 3.0 新增了 format() 函数，可以提供高级的格式化选项
print("【{}】今年【{}】岁".format(name, age))  # 【张三】今年【18】岁
# Python 3.6 f-string出现，使得格式化方法更加灵活，字符串前缀为 f，并使用 {} 评估值
print(f"【{name}】今年【{age}】岁")  # 【张三】今年【18】岁
```

其他用法： https://www.php.cn/faq/567934.html

### 2、字符串切片操作

```
s[i]
s[i:k]
s[i:k:k] // k是步长
```



### 2、其他用法

```
a + b  // 连接
s * b // b字符串重复次数的连接

a in b // a是否在b字符串中
a not in b

str() // 转换为字符串

Str.count(sub[, start[,end]]) // 返回子字符串sub在某个范围内非重叠出现的次数
Str.isalnum // 如果字符串中所有字符是字母或数字，且至少有一个字符，返回true
Str.isalpha() // 如果字符串中所有字符是字母，且至少有一个字符，返回true
Str.join(",") // 用固定字符串拼接字符串
Str.split(sep=None,maxsplit=-1) // 使用sep作为分隔符分割字符串
Str.startswith(prefix[,start[,end]])// 如果字符串以prefix开头，返回true
```

