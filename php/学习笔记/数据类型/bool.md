## main

[bool](https://php.p2hp.com/manual/zh/language.types.boolean.php) 仅有两个值，用于表达真（truth）值，不是 **`true`** 就是 **`false`**。



### [转换为布尔值](https://php.p2hp.com/manual/zh/language.types.boolean.php#language.types.boolean.casting)

要明确地将值转换成 [bool](https://php.p2hp.com/manual/zh/language.types.boolean.php)，可以用 `(bool)` 强制转换。通常这不是必需的，因为值在逻辑上下文中使用将会自动解释为 [bool](https://php.p2hp.com/manual/zh/language.types.boolean.php) 类型的值。更多信息请阅读[类型转换](https://php.p2hp.com/manual/zh/language.types.type-juggling.php)页面。

参见[类型转换的判别](https://php.p2hp.com/manual/zh/language.types.type-juggling.php)。

当转换为 [bool](https://php.p2hp.com/manual/zh/language.types.boolean.php) 时，以下值被认为是 **`false`**：

- [布尔](https://php.p2hp.com/manual/zh/language.types.boolean.php)值 **`false`** 本身
- [整型](https://php.p2hp.com/manual/zh/language.types.integer.php)值 `0`（零）
- [浮点型](https://php.p2hp.com/manual/zh/language.types.float.php)值 `0.0`（零）`-0.0`（零）
- 空[字符串](https://php.p2hp.com/manual/zh/language.types.string.php) `""`，以及[字符串](https://php.p2hp.com/manual/zh/language.types.string.php) `"0"`
- 不包括任何元素的[数组](https://php.p2hp.com/manual/zh/language.types.array.php)
- 原子类型 [NULL](https://php.p2hp.com/manual/zh/language.types.null.php)（包括尚未赋值的变量）
- 内部对象的强制转换行为重载为 bool。例如：由不带属性的空元素创建的 [SimpleXML](https://php.p2hp.com/manual/zh/ref.simplexml.php) 对象。

所有其它值都被认为是 **`true`**（包括 [资源](https://php.p2hp.com/manual/zh/language.types.resource.php) 和 **`NAN`**）。



## 相关链接

- 参考 https://php.p2hp.com/manual/zh/language.types.boolean.php