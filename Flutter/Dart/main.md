## main



### null safety

```dart
String? age = 20;// 表示age是一个可空类型
age=null;

String? getAge() {// 表示可以返回一个可空类型
	return null;
}
```

### 类型断言

```dart
String? name= "a";/ 
name=null;
print(str!.length);// 如果str不等于null会打印str的长度，如果等于null会抛出异常
// 正常写法需要哦按的str不等于null在计算长度
```



### 断言

其大致的作用是: if的缩写，如果assert(false)条件为false，那么久会抛出异常；并且这个只能在debug模式下有效。

```dart
String urlString = 'http://www.baidu.com';
assert(urlString.startsWith('https'), 'URL ($urlString) should start with "https".');
// Failed assertion: 'urlString.startsWith('https')': URL (http://www.baidu.com) should start with "https"
// 当urlString不是以https开头时，代码的执行会被打断
// 当urlString是以https开头时，代码会继续执行
```



### late 关键字 ,延迟初始化

```dart
class Web extends Person {
  late String sex; // 等同于 String？ sex
  Web(String name, int age, String sex) : super(name, age) {
    this.sex = sex;
  } 
}
```

 

## 相关wiki

- 库文件地址 https://pub.dev/packages
- 教程 https://www.yiibai.com/dart
- 教程2 https://www.tizi365.com/archives/165.html
- 语法教程 https://blog.csdn.net/m0_37293461/article/details/101099386
- 中文网站 https://www.dartcn.com/guides/language/language-tour
- 视频教程 https://www.bilibili.com/video/BV1S4411E7LY