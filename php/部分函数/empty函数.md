## empty
#### 检查一个变量是否为空

* 注意参数只能是变量
* <font color="red">尽量不要判断类变量，因为判断类变量时会有不同的判断结果</font>
* 当对一个不可见的对象属性使用 empty() 时， __isset() 方法如果存在的话，它将会被调用(所以这是相当于判断了__isset()返回的一个常量，返回true)。

``` 
class classA {
	private $a = 'a';
	protected $b = 'b';
	public $c = 'c';
 
	public function check() {
		var_dump(empty($obj->c)); // true
	}

	public function __isset($arg) {
		return isset($this->arg);
	}
 
 }

$obj = new classA();
$obj->check(); // 类内部调动，都返回true

var_dump(empty($obj->a)); // true
var_dump(empty($obj->b)); // true
var_dump(empty($obj->c));// false
// 外部调用，只有成员属性是public时，才返回正确判断结果
```






 