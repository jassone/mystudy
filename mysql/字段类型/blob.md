## BLOB
### 一、概念
BLOB (binary large object)二进制大对象，是一个可以存储**二进制数据**的容器。

在计算机中，BLOB常常是数据库中用来存储二进制文件的字段类型。

BLOB是一个大文件，典型的BLOB是一张图片或一个声音文件，由于它们的尺寸，必须使用特殊的方式来处理。

### 二、MySQL的四种BLOB类型：
MySQL中BLOB是个类型系列，包括：TinyBlob、Blob、MediumBlob、LongBlob，这几个类型之间的唯一区别是在存储文件的最大大小上不同。

| 类型 |  大小(单位：字节)|  
| --- | --- | 
|  TinyBlob| 最大255 |  
| Blob | 最大65K |  
| MediumBlob | 最大16M |  
| LongBlob |  最大4G |  

 