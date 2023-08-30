## InnoDB行记录格式
数据记录在磁盘上的存放方式也被称为 行格式 或者 记录格式 。
4种不同类型的 行格式 ，分别是 Compact 、 Redundant 、Dynamic 和 Compressed 行格式。

![e974699f12da4e9193b18a6690155255.png](https://pic.imgdb.cn/item/61ea33722ab3f51d914e1165.png)


| ROW_FORMAT |  支持索引前缀| 独立表空间压缩 |系统表空间压缩|
| --- | --- | --- |--- |
| COMPRESSED | 	3072字节 | 支持 |	不支持|
| DYNAMIC | 3072字节	 | 不支持 |	不支持|
| COMPACT | 768字节 | 	不支持 |	支持|
|  REDUNDANT| 	768字节 | 	不支持 |	支持|


在 msyql 5.7.9 及以后版本，默认行格式由innodb_default_row_format变量决定，它的默认值是`DYNAMIC`，也可以在 create table的时候指定ROW_FORMAT=DYNAMIC。

用户可以通过命令 
SHOW TABLE STATUS LIKE ‘table_name’ 
来查看当前表使用的行格式。

## 一、InnoDB行存储
InnoDB存储引擎记录是以行的形式存储的。这意味着页中保存着表中一行行的数据。这些页以树形结构组织，这颗树称为B树索引。表中数据和辅助索引都是使用B树结构。维护表中所有数据的这颗B树索引称为聚簇索引，通过主键来组织的。聚簇索引的叶子节点包含行中所有字段的值，辅助索引的叶子节点包含索引列和主键列。

##### 行记录最大长度
* 页大小（page size）为4KB、8KB、16KB和32KB时，行记录最大长度（maximum row length）应该略小于页大小的一半。

  **默认页大小为16KB**，行记录最大长度应该略小于8KB ，**因此一个B+Tree叶子节点最少有2个行记录**。
  
* 页大小为64KB时，行记录最大长度略小于16KB

##### CHAR(N)与VARCHAR(N)
N指的是`字符长度`，而不是Byte大小，在不同的编码下，同样的字符会占用不同的空间，如LATIN1（定长编码）和UTF8(变长编码)。

##### 变长列
在InnoDB中，变长列（variable-length column）可能是以下几种情况
* 长度不固定的数据类型，例如VARCHAR、VARBINARY、BLOB、TEXT等。
* **对于长度固定的数据类型，如CHAR，如果实际存储占用的空间大于768Byte，InnoDB会将其视为变长列**。
* **变长编码（如UTF8，GBK等）下的CHAR**

##### 隐藏列
* ROWID：没有显式定义主键或唯一非NULL的索引时，InnoDB会自动创建，6Byte大小。
* Transaction ID：事务ID
* Roll Pointer：回滚指针列

##### 行溢出
当行记录的长度没有超过行记录最大长度时，所有数据都会存储在当前页。

当行记录的长度超过行记录最大长度时，变长列（variable-length column）会选择外部溢出页（overflow page，一般是Uncompressed BLOB Page）进行存储。

说如果一个列中存储的数据小于8099个字节，那么该列就不会成为溢出列。如果表中有多个列，那么这个值更小。所以实际应用中要避免行益处。

* Compact + Redundant：保留前768Byte在当前页（B+Tree叶子节点），其余数据存放在溢出页。768Byte后面跟着20Byte的数据，用来存储指向溢出页的指针
* Dynamic + Compressed：仅存储20Byte数据，存储指向溢出页的指针，这时比Compact和Redundant更高效，因为一个B+Tree叶子节点能存放更多的行记录

![overflow-page.png](https://pic.imgdb.cn/item/61ea63672ab3f51d91870e07.png)

## 二、Compact
MySQL 5.0引入，MySQL 5.1默认ROW_FORMAT

对比Redundant
* 减少了大约20%的空间(NULL标志位的使用)
* 在某些操作下会增加CPU的占用
* 在典型的应用场景下，比Redundant快

### 格式
|变长字段长度列表| NULL标志位 | 记录头信息 |ROWID|Transaction ID|Roll Pointer|列1	…|
| --- | --- | --- |--- |--- |--- |--- |

##### 变长字段长度列表
* 放置排序：逆序
* 条件
  a)VARCHAR、BLOB等
  b）变长编码下的CHAR
* 用2Byte存储的情况：需要用溢出页；最大长度超过255Byte；实际长度超过127Byte

##### NULL标志位
* 行记录中是否有NULL值，是一个位向量（Bit Vector）
* 可为NULL的列数量为N，则该标志位占用的CEILING(N/8)Byte
* 列为NULL时不占用实际空间

##### 记录头信息

##### char(M)的定义
char(M)要求至少占用M个字节，比如使用utf8字符集，char(10)存储数据占用10-30字节。

## 三、Redundant
MySQL 5.0之前的ROW_FORMAT
### 格式
| 字段偏移列表 | 记录头信息	ROWID | Transaction ID |Roll Pointer|列1	…|
| --- | --- | --- |--- |--- |

##### 字段偏移列表
* 按照列的顺序逆序放置
* 列长度小于255Byte，用1Byte存储
* 列长度大于255Byte，用2Byte存储

##### 记录头信息

##### NULL值处理
* 存储null值的字段是定长类型，null也占用空间
* 存储null值的字段是变长类型，null不占用空间

##### char(M)的定义
char(M)要求至少占用 M*一个字符最大占用字节 的字节数。比如utf8字符集，char(10)占用30个字节。

## 四、Dynamic
MySQL版本是5.7，它的默认⾏格式就Dynamic。

dynamic行格式，列存储是否放到off-page页，主要取决于行大小，它会把行中最长的那一列放到off-page，直到数据页能存放下两行。TEXT/BLOB列 <=40 bytes 时总是存放于数据页。这种方式可以避免compact那样把太多的大列值放到 B-tree Node，因为dynamic格式认为，只要大列值有部分数据放在off-page，那把整个值放入都放入off-page更有效。

## 五、Compressed
compressed 物理结构上与dynamic类似，但是对表的数据行使用zlib算法进行了压缩存储。在long blob列类型比较多的情况下用，可以降低off-page的使用，减少存储空间（一般40%左右），但要求更高的CPU，buffer pool里面可能会同时存储数据的压缩版和非压缩版，所以也多占用部分内存。

