## mysql
## 工作流程
![20220120115600.jpg](https://pic.imgdb.cn/item/61e8dd5b2ab3f51d9107eb3e.jpg)

![v2-98c7bf713122ddd02352bea0578e42c2_r.jpeg](https://pic.imgdb.cn/item/61ee6b892ab3f51d914abad7.jpg)

人们把 连接管理、查询缓存、语法解析、查询优化 这些并不涉及真实数据存储的功能划分为MySQL server的功能，把真实存取数据的功能划分为 存储引擎 的功能。各种不同的存储引擎向上边的 MySQL server 层提供统一的调用接口（也就是`存储引擎API`），包含了几十个底层函数。

## 基础
* 默认端口3306
* 存储引擎是挂靠到表上的

### 连接方式
* tcp连接
* unix socket 套接字连接