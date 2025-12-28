## main

### 转换到 **`null`**

使用 `(unset) $var` 将一个变量转换为 [null](https://php.p2hp.com/manual/zh/language.types.null.php) 将*不会*删除该变量或 unset 其值。仅是返回 **`null`** 值而已。

- 本特性自 PHP 7.2.0 起*废弃*，并且自 PHP 8.0.0 起被*移除*。 强烈建议不要使用本特性。





## 相关函数

- [is_null()](https://php.p2hp.com/manual/zh/function.is-null.php)

  

## 其他

- 未定义和 [unset()](https://php.p2hp.com/manual/zh/function.unset.php) 的变量都将解析为值 **`null`**。

  

## 相关链接

- 参考 https://php.p2hp.com/manual/zh/language.types.null.php