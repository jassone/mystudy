## yield

## 什么是yield
### 1、一个demo
**正常情况下**
```php
function createRange($number){
    $data = [];
    for($i=0;$i<$number;$i++){
        $data[] = time();
    }
    return $data;
}

$result = createRange(10); // 这里调用上面我们创建的函数
foreach($result as $value){
    sleep(1);//这里停顿1秒，我们后续有用
    echo $value.'<br />';
}
```
**这时候打印出来的时间戳是一样的。**

我们注意到，如果传参为1000万，for循环就需要执行1000万次。且有1000万个值被放到 \$data 里面，而\$data数组在是被放在内存内。所以，在调用函数时候会占用大量内存。这里，生成器就可以大显身手了。

**用了yield**

```php
function createRange($number){
    for($i=0;$i<$number;$i++){
        yield time(); // 这里也可以是自定义的func
    }
}

$result = createRange(10); // 这里调用上面我们创建的函数
foreach($result as $value){
    sleep(1);
    echo $value.'<br />';
}
```
**这时候打印出来的时间戳是递增1秒的(这里的间隔一秒其实就是 sleep(1) 造成的后果)。**

使用生成器时： createRange 的值不是一次性快速生成，而是依赖于 foreach 循环。 **foreach 循环一次， for 执行一次**。

我们来还原一下代码执行过程。
* 首先调用 createRange 函数，传入参数10，但是 for 值执行了一次然后停止了，并且告诉 foreach 第一次循环可以用的值。
* foreach 开始对 $result 循环，进来首先 sleep(1) ，然后开始使用 for 给的一个值执行输出。
* foreach 准备第二次循环，开始第二次循环之前，它向 for 循环又请求了一次。
* for 循环于是又执行了一次，将生成的时间戳告诉 foreach .
* foreach 拿到第二个值，并且输出。由于 foreach 中 sleep(1) ，所以， for 循环延迟了1秒生成当前时间.

### 2、原理
yield是生成器所需要的关键字,必须在函数内部,有yield的函数叫做"生成器函数"。该函数将返回一个可遍历的对象（一个继承了Iterator的生成器）。

**生成器yield**关键字不是返回值，他的专业术语叫`产出值`，只是`生成一个值`。

那么代码中 foreach 循环的是什么？其实是PHP在使用生成器的时候，会返回一个 **Generator 类的对象**。

```php
Generator implements Iterator {
    //返回当前产生的值
    public mixed current ( void )
    //返回当前产生的键
    public mixed key ( void )
    //生成器继续执行
    public void next ( void )
    //重置迭代器,如果迭代已经开始了，这里会抛出一个异常。
    public void rewind ( void )
    //向生成器中传入一个值，当前yield接收值，然后继续执行下一个yield
    public mixed send ( mixed $value )
    //向生成器中抛入一个异常
    public void throw ( Exception $exception )
    //检查迭代器是否被关闭,已被关闭返回 FALSE，否则返回 TRUE
    public bool valid ( void )
    //序列化回调
    public void __wakeup ( void )
    //返回generator函数的返回值，PHP version 7+
    public mixed getReturn ( void )
}
``` 

foreach 可以**对该对象进行迭代**，每一次迭代，PHP会通过 Generator 实例计算出下一次需要迭代的值。这样 foreach 就知道下一次需要迭代的值了。

而且，在运行中 for 循环执行后，会立即停止。等待 foreach 下次循环时候再次和 for 索要下次的值的时候，循环才会再执行一次，然后立即再次停止。直到不满足条件不执行结束。

这也解释了为什么createRange叫做迭代生成器, 因为它返回一个迭代器, 而这个迭代器实现了Iterator接口.

调用迭代器的方法一次, 其中的代码运行一次。即可以简单把yield理解为暂时停止函数的执行，等待外部的激活从而再次执行（执行到当前暂停点结束当前这轮执行）。虽然看起来确实像那么回事，但不建议这么理解，因为他本身是返回一个迭代器对象，其返回值是可以被用于迭代的。

### 3、特点
* yield只能用于函数内部，在非函数内部运用会抛出错误。
* 如果函数包含了yield关键字的，那么函数执行后的返回值永远都是一个Generator对象。
* 如果函数内部同事包含yield和return 该函数的返回值依然是Generator对象，但是在生成Generator对象时，return语句后的代码被忽略。
* Generator类实现了Iterator接口。
* 可以通过返回的Generator对象内部的方法，获取到函数内部yield后面表达式的值。
* 可以通过Generator的send方法给yield 关键字赋一个值。
* 一旦返回的Generator对象被遍历完成，便不能调用他的rewind方法来重置
* Generator对象不能被clone关键字克隆

### 4、优点
* 生成器会对PHP应用的性能有非常大的提高
* PHP代码运行时节省大量的内存,可以用于内存优化。
* 比较适合计算大量的数据
* 通过继承Iterator接口也可以，性能开销和复杂性大大降低。

## 二、其他用法属性
### 1、不能被转换为字符串
```php
function getValues() {
    yield 'value';
}

echo getValues();
// PHP Fatal error:  Uncaught Error: Object of class Generator could not be converted to string 
//类生成器的对象不能被转换成字符串， 
```
### 2、“yield” & “return” 的区别
前面的错误意味着 getValues() 方法不会如预期返回一个字符串，让我们检查一下他的类型：
```php
function getValues() {
    yield 'value';
}
var_dump(getValues()); // class Generator#1 (0) {}
```
生成器 类实现了 生成器 接口， 这意味着你必须遍历 getValue() 方法来取值：

```php
foreach (getValues() as $value) {
   echo $value;
}

// 使用变量也是好的
$values = getValues();
foreach ($values as $value) {
   echo $value;
}
```

## 三、应用场景
### 1、读取大的文件
比如可以每次读取一行，每次被加载到内存中的文字只有一行，大大的减小了内存的使用。

## 四、wiki
* https://segmentfault.com/a/1190000022612397
* https://segmentfault.com/a/1190000022754223
* https://segmentfault.com/a/1190000022700409
* https://zhuanlan.zhihu.com/p/33055070
