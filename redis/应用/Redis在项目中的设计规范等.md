## Redis在项目中的设计规范、内存保障与安全选项

## 一、redis设计规范
### 1、key的设计
##### a) 可读性和可管理型
业务名:实体名:id

##### b) 简洁性
减少key的长度，建议不超过44个字节。此时是embstr编码的方式来保存，效率高。

##### c) 不要包含特俗字符
建议英文+数字

## 二、redis安全建议
### 1、redsi不要在外网访问，禁止bind 0.0.0.0

### 2、更改redis端口，不要用6379

### 3、redis使用非root启动
分配个专用账号去管理。

### 4、redis没有设置密码或者弱密码，不要与登陆密码相同
* 客户端访问服务器时的requirepass密码
* 主从同步时的masterauth密码

### 5、定期备份-执行save命令

### 6、配置好linux防火墙规则
* 暴露最小必要的ip地址
* 放行最小范围的端口

## 三、redis内存占用评估
![20220420203204.jpg](https://pic.imgdb.cn/item/625ffd23239250f7c542c5f6.jpg)

redis内存统计
![20220420203521.jpg](https://pic.imgdb.cn/item/625ffde9239250f7c5448896.jpg)

内存空间配置
![20220420203743.jpg](https://pic.imgdb.cn/item/625ffee4239250f7c54668c9.jpg)

内存回收配置
![20220420204009.jpg](https://pic.imgdb.cn/item/625ffff0239250f7c5486d4f.jpg)