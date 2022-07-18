## json

JSON，几乎每个软件开发者都接触使用过，但大都只会使用而已，究竟JSON是什么，怎么来的，有哪些应用，可能是一知半解，今天我们就罗列几个JSON重要的知识点。

JSON是一个序列化的对象或数组。

## 一、JSON定义

**JavaScript对象表示法/标记(JavaScript Object Notation)， 是一种轻量级的数据交换格式。**

**JSON 语法是 JavaScript 对象表示法语法的子集。是基于javascript的对象字面量(因为JSON里面没有类似JavaScript对象方法/函数。字面量：指字面意思和要表达的意思是完全一致的值)。**

JSON是一项技术标准，采用完全独立于程序语言的文本格式。JavaScript对象表示法，去掉“JavaScript”，就是对象表示法，所以忘掉“JavaScript”吧，我们使用的是一种基于对象表示法的数据交换格式，这使JSON成为理想的数据交换语言。

JSON数据文件的扩展名为.json。其实只要数据格式符合JSON规范，不一样要*.json的文件，文本格式皆可。

## 二、JSON的数据格式/类型

JSON 采用"名称 / 值对"的数据格式(A collection of name/value pairs)。不同的编程语言中，它被理解为对象（object），纪录（record），结构（struct），字典（dictionary），哈希表（hash table），有键列表（keyed list），或者关联数组 （associative array）。 值的有序列表（An ordered list of values）。在大部分语言中，它被实现为数组（array），矢量（vector），列表（list），序列（sequence）。

这些都是常见的数据结构。目前，绝大部分编程语言都以某种形式支持它们。这使得在各种编程语言之间交换同样格式的数据成为可能。

JSON包含六个构造字符(  { } [ ] : ,)、字符串、数字和三个字面名(true,false,null)。

JSON的数据结构有如下6种：
- number：和JavaScript的number完全一致；
- boolean：就是JavaScript的true或false；(必须为小写)
- string：就是JavaScript的string；
- null：就是JavaScript的null；(必须为小写)
- array：就是JavaScript的Array表示方式——[]；
- object：就是JavaScript的{ ... }表示方式。

## 三、如何在JavaScript中使用JSON

由于JSON非常简单，很快就风靡Web世界，并且成为ECMA标准。而且几乎所有编程语言都有解析JSON的库。JSON 是 JavaScript 原生格式，这意味着在 JavaScript 中处理 JSON 数据不需要任何特殊的 API 或工具包。下面就以JavaScript为例，看下如何使用JSON。其他语言则需要安装扩展，这里不再举例。

- 把任何JavaScript对象变成JSON，就是把这个对象序列化成一个JSON格式的字符串，这样才能够通过网络传递给其他计算机。
- 如果我们收到一个JSON格式的字符串，只需要把它反序列化成一个JavaScript对象，就可以在JavaScript中直接使用这个对象了。

JSON.parse是将json格式的字符串转换成json对象。
```javascript
var str ='{"name":"张三","age":16,"msg":["a","b"]}';
var json = JSON.parse(str);
console.log("name:" + json.name);
console.log("msgLen:" + json.msg.length);
#输出
name:张三
msgLen:2
```
JSON.stringify将json对象转换成json格式的字符串。
```javascript
var json = {"name":"张三","age":16,"msg":["a","b"]};
var str = JSON.stringify(json);
console.log("json:" + str);
console.log("jsonLen:" + str.length);
#输出
json:{"name":"张三","age":16,"msg":["a","b"]}
jsonLen:38
```

注意下面一种情况
```javascript
var user = {
 name : "test",
 'age' : 'test'
};
```
这是对象，不是json。

> JSON 转义

需要转义的字符
- "
- \
- \/(正斜线)
- \b(退格符)
- \f(换页符)
- \t(制表符)
- \n(换行符)
- \r(回车符)
- \u(后面跟着十六进制符号，如笑脸\u263A)

例如下面的双引号，前面必须加"\\"
```
{
  "title": "A \"book\""
}
```

## 四、JSON schema

JSON Schema 是一个允许你标注和验证JSON文档工具，其遵循Json规范，本身就是一个Json字符串，可以做API自动测试和验证客户端提交的数据。

详细说明可以浏览官网：[http://json-schema.org/](http://json-schema.org/)

这里以PHP为例，演示一下具体使用（项目来源地址:[https://github.com/justinrainbow/json-schema](https://github.com/justinrainbow/json-schema)
）
data.json
```json
{
  "name":"jack",
  "age":1000,
  "height":1
}
```
schema.json
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "format": "regex",
      "pattern": "[a-z]+"
    },
    "age": {
      "type": "integer",
      "default": 0,
      "minimum": 0,
      "maximum": 100
    },
    "height": {
      "type": "integer",
      "default": 0,
      "minimum": 0,
      "maximum": 2
    }
  }
}
```
index.php
```php
<?php
require __DIR__ . '/../vendor/autoload.php';

$data = json_decode(file_get_contents('data.json'));

// Validate
$validator = new JsonSchema\Validator();
$validator->check($data, (object) array('$ref' => 'file://' . realpath('schema.json')));

if ($validator->isValid()) {
    echo "The supplied JSON validates against the schema.\n";
} else {
    echo "JSON does not validate. Violations:</br>";
    foreach ($validator->getErrors() as $error) {
        echo sprintf("[%s] %s\n", $error['property'], $error['message']) . '</br>';
    }
}
?>
```
输出(这里age验证不通过)
JSON does not validate. Violations:
[age] Must have a maximum value of 100 

## 最后贴几个好用的在线工具网站

在线语法验证解析
[https://jsoneditoronline.org/](https://jsoneditoronline.org/)
[https://jsonlint.com](https://jsonlint.com/)
[https://jsonformatter.curiousconcept.com/](https://jsonformatter.curiousconcept.com/)

JSON schema 在线验证
[https://www.jsonschema.net/](https://www.jsonschema.net/)(可以自动生成JSON schema)
[https://jsonschemalint.com/](https://jsonschemalint.com/)
[https://www.jsonschemavalidator.net/](https://www.jsonschemavalidator.net/)

JSON在线搜索
[http://jsonpath.com/](http://jsonpath.com/)
[https://jqplay.org/](https://jqplay.org/)


将JSON转换为HTML
[http://tryhandlebarsjs.com/](http://tryhandlebarsjs.com/)



 