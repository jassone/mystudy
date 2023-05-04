## main

## 一、基础知识

Kubernetes也称为K8S，其中8是代表中间“ubernete”的8个字符，是Google在2014年开源的一个容器编排引擎，用于自动化容器化应用程序的部署、规划、扩展和管理，它将组成应用程序的容器分组为逻辑单元，以便于管理和发现，用于管理云平台中多个主机上的容器化的应用，Kubernetes 的目标是让部署容器化的应用简单并且高效，很多细节都不需要运维人员去进行复杂的手工配置和处理；

通过Kubernetes你可以：

- 快速部署应用
- 快速扩展应用
- 无缝对接新的应用功能
- 节省资源，优化硬件资源的使用

### 1、k8s系统架构

从系统架构来看，k8s分为2种节点：

- Master 控制节点，做为指挥官；
- Node 工作节点，干活的

##### a) Master节点组成



 ##### b） node节点组成



 ### 2、k8s逻辑架构
从逻辑架构上看，k8s分为Pod、Controller、Service 。

##### a) POD




##### b) Controller

用来管理POD。控制器的种类有很多：



- Deployment：推荐使用，功能更强大，包含了RS控制器
- DaemonSet：保证所有的Node上有且只有一个Pod在运行
- StatefulSet：有状态的应用，为Pod提供唯一的标识，它可以保证部署和scale的顺序

##### c）Service

理解三个不同的ip：NodeIP、CluterIP、POD IP。



## 相关文档

- 学习资源 https://github.com/guangzhengli/k8s-tutorials
- 中文 https://kubernetes.io/zh-cn/
- ApiServer接口文档 https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/
- 名词对比 https://cloud.tencent.com/document/product/457/45612
- https://www.apiref.com/kubernetes-zh/index.html





