## char原理

todo

总结来看，char脱离了原有字符的语意，理解为code unit更加合适。
**一个char等于一个code unit，而不等于一个字符**。
当表示BMP字符，utf-16使用一个code unit，此种情况Java中的char才具有字符的语意。

String本意为char类型的序列，如今理解为code unit的序列也许更合适。有点字符串不是字符串的魔幻感觉，应该叫作code unit串。

## 相关wiki

- Java中关于Char存储中文到底是2个字节还是3个还是4个？https://www.zhihu.com/question/279539793