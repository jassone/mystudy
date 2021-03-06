## RESTful API接口

* REST 是 Representational State Transfer的缩写，如果一个架构符合REST原则，就称它为RESTful架构
* RESTful 架构可以充分的利用 HTTP 协议的各种功能，是 HTTP 协议的最佳实践
* RESTful API 是一种软件架构风格、设计风格，可以让软件更加清晰，更简洁，更有层次，可维护性更好

## 一、接口命名规范和语义格式
### 1、基于Http协议，URL对外暴露
##### a) 请求 = 动词 + 宾语 + 过滤信息
* 动词 使用五种 HTTP 方法，对应 CRUD 操作。
* 宾语 URL 应该全部使用名词复数，可以有例外，比如搜索可以使用更加直观的 search(带上版本信息v1/search) 。
* 过滤信息（Filtering） 如果记录数量很多，API应该提供参数，过滤返回结果。 ?limit=10 指定返回记录的数量 ?offset=10 指定返回记录的开始位置。

### 2、使用XML或JSON格式定义

### 3、根据不同行为使用不同请求方式(GET,POST,DELEDT...)

## 二、接口设计规范和注意事项
### 1、接口要保证幂等性设计

### 2、标准化的响应结果集

### 3、接口设计采用无状态方案
错误案例：接口服务端维护了sesseion，可能不同服务器最终接口执行结果不一样。

可行方案
* 客户端存储用户数据
* 服务端分布式存储

## 三、REST缺点
### 1、基于文本的低效消息协议
因为是**基于http协议**的，而文本带有字符集，以及编码等转换，解析效率不高。

### 2、应用程序之间缺乏强类型接口
客户端和服务端字段不统一。本地不能对请求进行检查，只能调用接口后才能发现问题。

### 3、难以强制实施
对某些约定不能强制约束。





