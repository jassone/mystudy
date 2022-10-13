## String

String不是java中的基础类型，但是可以当成基础类型，便于理解。String也不是保留字。

### 1、string是引用类型

```java
String s = "hello";
```

为什么String是引用类型呢，其实它也是一个类。

```java
String s = new String("aaa");
```

引用类型的变量类似于C语言的指针，它内部存储一个“地址”，指向某个对象在内存的位置。

**String.length()返回的是code unit的个数，而不是字符的个数，也不是占用字节的个数！！！**

```java
String name = "jack";  //Latin-1编码, 4个字节, 4个code unit       
String name1 = "小明";   //utf-16编码, 4个字节, '小明'两个字符属于BPM范畴，2个code unit        
String name2 = "jack小明";   //utf-16编码, 12个字节，6个code unit         System.out.println(name.length()); //4       
System.out.println(name1.length()); //2       
System.out.println(name2.length()); //6
```

**统计String中的字符(code point)数，请使用String.codePointCount()方法!**

### 2、字符串不可变

```java
String s = "hello";
s = "world";
```

字符串的不可变是指字符串内容不可变。至于变量，可以一会指向字符串`"hello"`，一会指向字符串`"world"`。上面s只是指向了另外一个内存地址而已，原来的"hello"无法通过s进行访问。

```java
String s = "hello";
String t = s;
s = "world"; //此时t还是hello
```

同理，数组元素中的字符串也不能修改。

### 3、null

引用类型的变量可以指向一个空值`null`，它表示不存在，即该变量不指向任何对象。例如：

```java
String s1 = null; // s1是null
String s2 = s1; // s2也是null
```

