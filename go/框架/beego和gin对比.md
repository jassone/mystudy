## beego和gin对比

## 一、mvc

beego支持(典型的MVC框架)，gin不完全支持。

### 1、beego-Model

- 支持go的所有类型存储
- 更简单的CURD风格
- 完整实现健壮稳定的ORM

### 2、beego-View

- 支持静态文件的处理
- 支持模板的处理
- 支持模板分页的处理

### 3、beego-Controller

- 路由功能
- 控制器函数
- CSRF过滤器
- Session
- 错误处理 & 日志处理

### 4、gin

- HTML渲染和模板
- 静态文件服务
- 路由

## 二、路由

beego支持路由，gin支持。

### 1、beego

- RESTful Controller路由（那种普通的）
- 正则路由
- 不支持组路由

### 2、gin

- RESTful Controller路由（那种普通的）
- 不支持正则路由  
- 支持组路由

## 三、session

beego支持session，gin不支持

### 1、beego

对后端引擎的支持： Menory,file,Mysql,Redis

### 2、gin

需要另外安装包，如github.com/astaxie/session



## 四、两个框架如何选择

### 1、beego在业务方面较gin支持的更多

- 在业务更加复杂的项目中，使用beego
- 在需要快速开发的项目中，使用beego
- 在初始项目中，使用beego

### 2、gin在性能方面较beego更好

- 当某个接口的性能遭到较大挑战的时候，考虑用gin重写
- 如果项目规模不大，业务相对简单，使用gin







