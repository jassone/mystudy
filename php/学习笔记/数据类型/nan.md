## nan - not a number

某些数学运算会产生一个由常量 **`NAN`** 所代表的结果。此结果代表着一个在浮点数运算中未定义或不可表述的值。任何拿此值与其它任何值（除了 **`true`**）进行的松散或严格比较的结果都是 **`false`**。

由于 **`NAN`** 代表着任何不同值，不应拿 **`NAN`** 去和其它值进行比较，包括其自身，应该用 [is_nan()](https://php.p2hp.com/manual/zh/function.is-nan.php) 来检查。