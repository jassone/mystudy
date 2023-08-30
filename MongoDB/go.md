

在GO中使用BSON对象
MongoDB中的JSON文档以称为BSON（二进制编码的JSON）的二进制表示形式存储。与其他将JSON数据存储为简单字符串和数字的数据库不同，BSON编码扩展了JSON表示形式，例如int，long，date，float point和decimal128。这使应用程序更容易可靠地处理，排序和比较数据。Go Driver有两种系列用于表示BSON数据：D系列类型和Raw系列类型。
D系列包括四种类型：

- D：BSON文档。此类型应用在顺序很重要的场景下，例如MongoDB命令。
- M：无序map。除不保留顺序外，与D相同。
- A：一个BSON数组。
- E：`D`中的单个元素。





## 相关文档

- go https://pkg.go.dev/go.mongodb.org/mongo-driver/mongo