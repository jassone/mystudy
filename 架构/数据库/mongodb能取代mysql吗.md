## mongodb能取到mysql吗

> 某人之言

现在mongodb 4.2（支持跨节点transaction）之后，我敢说，已经没有什么sql系关系型数据能做，mongodb做不了的。

### 1、mongodb的优势：
* 项目开发效率【远高于】sql（除非从来不改需求）
* 基本没有类似“sql注入”的安全隐患。
* 表结构的管理成本【远低于】sql（除非从来不改需求）
* 数据规模增大后的拆分成本【远低于】sql（除非项目越做越凉）
* 传输性能比sql高（sql啥都要编码成纯文本）
* 语法简单易学

### 2、它和sql比的“缺点”
* nosql系缺乏范式理论。
    然而这些理论：
    让你的数据库能最优化地利用存储空间——然而并没人在乎；

* 试图保证数据有效性。
    并不支持多态，所以基本没用；

* mongodb不支持行锁
    然而行锁并不能简单无脑地帮你解决并发安全问题（竞争冒险），使用全表锁就更不现实了。
    虽然行锁能解决一部分并发安全问题，但是它还是不能提供一套解决此类问题的泛式。
    想要保证并发安全，很多时候还是在应用层来处理（比如该分类分类、该排队排队）
    也就是说虽然mongodb也不能完美的包办并发安全的问题，但是换成sql也没什么卵用

* mongodb对复杂的联表查询支持不高，比如递归和cross join。
    然而你可以在后端手动拼接。总结就是，mongodb解决不了的，sql也解决不了，但sql解决不了的，mongodb未必不能解决。
    **sql说到底是面向用户的历史遗留产物，并不是给后端用的**。

* mongodb不能修改主键的值。
* mongodb的图形界面管理工具太贵了，像mysql workbench这种都是免费的。