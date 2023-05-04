## Docker

Docker 是一个应用打包、分发、部署的工具。

- **打包**：就是把你软件运行所需的依赖、第三方库、软件打包到一起，变成一个安装包
- **分发**：你可以把你打包好的“安装包”上传到一个镜像仓库，其他人可以非常方便的获取和安装
- **部署**：拿着“安装包”就可以一个命令运行起来你的应用，自动模拟出一摸一样的运行环境，不管是在 Windows/Mac/Linux。

### 1、Docker 通常用来做什么

- 应用分发、部署，方便传播给他人安装。特别是开源软件和提供私有部署的应用
- 快速安装测试/学习软件，用完就丢（类似小程序），不把时间浪费在安装软件上。例如 Redis / MongoDB / ElasticSearch
- 多个版本软件共存，不污染系统，例如 Redis4.0，Redis5.0
- Windows 上体验/学习各种 Linux 系统

## 一、基础知识

### 1、版本

目前使用 Docker 基本上有两个选择：**Docker Desktop（桌面版）** 和 **Docker Engine（服务器版）**。

- Docker Desktop是商业产品，有一些自己的、非通用的东西，不利于我们后续的 Kubernetes 学习。第二个，它只是对个人学习免费，受条款限制不能商用，我们在日常工作中难免会“踩到雷区”。而实际上Docker Desktop内部包含了Docker Engine ，Docker Engine是其核心组件之一。
- Docker Engine 完全免费，但只能在 Linux 上运行，只能使用命令行操作，缺乏辅助工具，需要我们自己动手 DIY 运行环境。也是现在各个公司在生产环境中实际使用的 Docker 产品。

### 2、架构

![25b87755dfe.jpeg](https://pic1.imgdb.cn/item/6443f0d90d2dde577780cb22.webp)

Docker Engine 是典型的客户端 / 服务器（C/S）架构（**目的就是解耦**），命令行工具 Docker 直接面对用户，后面的 Docker daemon 和 Registry 协作完成各种功能。所以真正干活的其实是默默运行在后台的 Docker daemon，而我们实际操作的命令行工具“docker”只是个“传声筒”的角色。



## 二、相关资源

### 1、监控工具

cAdvisor是一款强大的 Docker Container 监控工具

## 2、相关软件安装教程

- docker安装golang https://blog.csdn.net/qq_39688201/article/details/125085264
- docker安装ngixn https://blog.csdn.net/BThinker/article/details/123507820
- docker安装clickhouse https://www.cnblogs.com/it1042290135/p/16202478.html

## 三、相关文档

### 1、官方

- 容器软件搜索 https://hub.docker.com/
- 命令行文档 https://docs.docker.com/engine/reference/run/

### 2、其他

- 使用教程-赞 https://docker.easydoc.net/doc/81170005/cCewZWoN/lTKfePfP
- 教程2 https://blog.csdn.net/weixin_45043334/category_11863858.html
- 卸载docker https://blog.csdn.net/qq_44028724/article/details/123806192
- Docker 和 Kubernetes 从听过到略懂：给程序员的旋风教程 https://juejin.cn/post/6844903650611953677
-  为什么Kubernetes天然适合微服务？ https://juejin.cn/post/6844903591774289934