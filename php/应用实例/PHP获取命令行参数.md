## PHP获取命令行参数

## 一、PHP获取命令行参数的方法

### 1、脚本中直接访问\$argv, \$argc两个全局变量：

* \$argv: 命令行实际参数的数组
* \$argc: 命令行参数的个数

比如现在有一个arg_test.php的文件：

``` sh
var_dump($argc);
var_dump($argv);

在命令行以下面的命令启动：php arg_test.php -t 800，输出如下：

int(5)
array(5) {
  [0]=>
  string(12) "arg_test.php"
  [1]=>
  string(2) "-t"
  [2]=>
  string(3) "800"
}

```

一个空格区分一个参数，包括当前的脚本名称，都会放在$argv中，当时这种方式不便于处理-t这样的参数，需要自己进行解析。

### 2、**通过getopt()函数（详见getopt函数）**

##### 命令行输入和输出

有的时候，我们需要在PHP脚本中检测用户的输入，以及输出一些提示信息给用户。这就需要用到PHP定义的标准输入输出流：

STDIN: 一个已打开的指向stdin的流
STDOUT: 一个已打开的指向stdout的流
STDERR: 一个已打开的指向stderr的流

可以像操作文件句柄一样操作这些标准流，而且不需要手动关闭，PHP会自行关闭。可以发现PHP对这三个常量的定义如下：

define(‘STDIN’, fopen(‘php://stdin’, ‘r’));
define(‘STDOUT’, fopen(‘php://stdout’, ‘w’));
define(‘STDERR’, fopen(‘php://stderr’, ‘w’));

显然这三个常量已经是打开的文件句柄，可以直接进行操作，下面是一个实例：

```php
<?php
// 像写入文件一样，将内容显示到控制台
fwrite(STDOUT, 'please input a number between 1 and 10: ');
input = fgets(STDIN);  // 从控制台读取输入continue = true;
// 轮询输入，知道正确为止
while (continue) {
    if (input < 1 || input>10) {
        // 输出到错误流
        fwrite(STDERR, 'you input number is wrong: ' .input);
    } else {
        fwrite(STDOUT, 'you input number is correct: ' . input);continue = false;
    }
    $input = fgets(STDIN);  // 从控制台读取输入
}
PHP
交互过程如下，其中输出到STDERR的内容会标红显示：

please input a number between 1 and 10: 12
you input number is wrong: 12
34
you input number is wrong: 34
9
you input number is correct: 9

```

## 二、运行 DEMO

### 1、输出目录区别

```sh
php 文件内容
echo getcwd();

➜  www git:(master) ✗ pwd
/Users/bian/webdata/www

// 注意以下两种方式输出区别
➜  www git:(master) ✗ php test/arg1.php 
/Users/bian/webdata/www%  
                                                         ➜  www git:(master) ✗ php-cgi test/arg1.php
X-Powered-By: PHP/7.4.22
Content-type: text/html; charset=UTF-8

/Users/bian/webdata/www/test%   

```

### 2、输出内容区别

```sh
php 文件内容
print_r($argv);

➜  www git:(master) ✗ php-cgi test/arg1.php  1 2
X-Powered-By: PHP/7.4.22
Content-type: text/html; charset=UTF-8

// CGI SAPI 模式中 $argv 打印的内容竟然是头信息，而不是具体的参数信息。
// 毕竟CGI SAPI模式本来就是为Web服务器提供的接口，所以它接收的是post 、 get这类的参数
// 而不是命令行的参数

➜  www git:(master) ✗ php test/arg1.php  1 2
<pre>Array
(
    [0] => test/arg1.php
    [1] => 1
    [2] => 2
)
// CLI SAPI 模式下我们正常获得了参数内容，
// 并且 $argv[0] 始终保存的是当前运行文件及路径

```

### 3、获取'-x'内容

```sh
➜  www git:(master) ✗ php -r 'var_dump($argv);' -- -h
array(2) {
  [0]=>
  string(19) "Standard input code"
  [1]=>
  string(2) "-h"
}

// 需要传递带 - 符号的内容时，需要先给一个 -- 参数列表分隔符。
// 这是因为 -xxx 的内容会让 php 命令认为这是一个命令选项而不是参数，
// 所以我们添加一个分隔符就可以让分隔符之后的参数内容原样传递进代码中
```

### 4、交互式运行

```sh
➜  www git:(master) ✗ php -a
Interactive shell

php > $a=1;
php > echo $a;
1
php > 
```

### 5、查看某个文件

**可以看成是像前端的代码压缩一样的能力**。 打印内容是去除掉所有注释和空白行的结果。

```sh
➜  test git:(master) ✗ php -w arg1.php                    
<?php  echo '<pre>'; print_r($argv); print_r($argc); die; $options = getopt('a:b:cde'); echo '<pre>'; print_r($options); %   
```

### 6、管道读取输入

```sh
➜  test git:(master) ✗ cat arg1.php |  php -r "print file_get_contents('php://stdin');"
<?php 
print_r($argv);
```
