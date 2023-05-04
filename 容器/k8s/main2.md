## 组件

### 1、master

- apiserver
- etcd
- scheduler
- controller-manager

这 4 个组件也都被容器化了，运行在集群的 Pod 里，查看 kubectl get pod -n kube-system

### 2、worker

- kubelet
- kube-proxy
- container-runtime

这 3 个组件中只有 kube-proxy 被容器化了，而 kubelet 因为必须要管理整个节点，容器化会限制它的能力，所以它必须在 container-runtime 之外运行。

## 插件

- DNS
- Dashboard
- Flannel ，Calico、Cilium  网络插件
- Metrics Server



## 对象

Kubernetes 版本支持的所有Api对象查看： kubectl api-resources

比如 ConfigMap、Pod、Service



- “在线业务”类型的应用有很多，比如 Nginx、Node.js、MySQL、Redis 等等，一旦运行起来基本上不会停，也就是永远在线。

- “离线业务”类型的应用也并不少见，它们一般不直接服务于外部用户，只对内部用户有意义，比如日志分析、数据建模、视频转码。

  “离线业务”也可以分为两种。一种是“**临时任务**”，跑完就完事了，下次有需求了说一声再重新安排；另一种是“**定时任务**”，可以按时按点周期运行，不需要过多干预。



### Deployment

“**Deployment**”，顾名思义，它是专门用来部署应用程序的，能够让应用永不宕机，多用来发布无状态的应用，是 Kubernetes 里最常用也是最有用的一个对象。

先看 replicas 字段。它的含义比较简单明了，就是“副本数量”的意思，也就是说，指定要在 Kubernetes 集群里运行多少个 Pod 实例。

有了这个字段，就相当于为 Kubernetes 明确了应用部署的“期望状态”，Deployment 对象就可以扮演运维监控人员的角色，自动地在集群里调整 Pod 的数量。



### DaemonSet

**DaemonSet**，它会在 Kubernetes 集群的每个节点上都运行一个 Pod，就好像是 Linux 系统里的“守护进程”（Daemon）。

DaemonSet 的目标是在集群的每个节点上运行且仅运行一个 Pod，就好像是为节点配上一只“看门狗”，忠实地“守护”着节点，这就是 DaemonSet 名字的由来。

有一些业务比较特殊，它们不是完全独立于系统运行的，而是与主机存在“绑定”关系，必须要依附于节点才能产生价值，比如说：

- 网络应用（如 kube-proxy），必须每个节点都运行一个 Pod，否则节点就无法加入 Kubernetes 网络。
- 监控应用（如 Prometheus），必须每个节点都有一个 Pod 用来监控节点的状态，实时上报信息。
- 日志应用（如 Fluentd），必须在每个节点上运行一个 Pod，才能够搜集容器运行时产生的日志数据。
- 安全应用，同样的，每个节点都要有一个 Pod 来执行安全审计、入侵检查、漏洞扫描等工作。

虽然我们没有指定 DaemonSet 里 Pod 要运行的数量，**但它自己就会去查找集群里的节点，在节点里创建 Pod**。因为我们的实验环境里有一个 Master 一个 Worker，而 Master 默认是不跑应用的，所以 DaemonSet 就只生成了一个 Pod，运行在了“worker”节点上。

##### 污点（taint）和容忍度（toleration）来控制master节点是否运行某个应用



### 静态 Pod

“静态 Pod”非常特殊，它不受 Kubernetes 系统的管控，不与 apiserver、scheduler 发生关系，所以是“静态”的。

但既然它是 Pod，也必然会“跑”在容器运行时上，也会有 YAML 文件来描述它，而唯一能够管理它的 Kubernetes 组件也就只有在每个节点上运行的 kubelet 了。

“静态 Pod”的 YAML 文件默认都存放在节点的 /etc/kubernetes/manifests 目录下，它是 Kubernetes 的专用目录。

你可以看到，Kubernetes 的 4 个核心组件 apiserver、etcd、scheduler、controller-manager 原来都以静态 Pod 的形式存在的，这也是为什么它们能够先于 Kubernetes 集群启动的原因。

如果你有一些 DaemonSet 无法满足的特殊的需求，可以考虑使用静态 Pod，编写一个 YAML 文件放到这个目录里，节点的 kubelet 会定期检查目录里的文件，发现变化就会调用容器运行时创建或者删除静态 Pod。



### Service

Service，它是集群内部的负载均衡机制，用来解决服务发现的关键问题。

Service 的工作原理和 LVS、Nginx 差不多，Kubernetes 会给它分配一个静态 IP 地址，然后它再去自动管理、维护后面动态变化的 Pod 集合，当客户端访问 Service，它就根据某种策略，把流量转发给后面的某个 Pod。

每个节点上的 kube-proxy 组件自动维护 iptables 规则，客户不再关心 Pod 的具体地址，只要访问 Service 的固定 IP 地址，Service 就会根据 iptables 规则转发请求给它管理的多个 Pod，是典型的负载均衡架构。

Kubernetes 为 Service 对象自动分配了一个 IP 地址“10.96.240.115”，这个地址段是独立于 Pod 地址段的（比如第 17 讲里的 10.10.xx.xx）。而且 Service 对象的 IP 地址还有一个特点，它是一个“**虚地址**”，不存在实体，只能用来转发流量。

Service 确实用一个静态 IP 地址代理了两个 Pod 的动态 IP 地址。

顺便说一下，Kubernetes 也为每个 Pod 分配了域名，形式是“**IP 地址. 名字空间.pod.cluster.local**”，但需要把 IP 地址里的 . 改成 - 。比如地址 10.10.1.87，它对应的域名就是 10-10-1-87.default.pod。

##### 如何让 Service 对外暴露服务

Service 对象有一个关键字段“**type**”，表示 Service 是哪种类型的负载均衡。前面我们看到的用法都是对集群内部 Pod 的负载均衡，所以这个字段的值就是默认的“**ClusterIP**”，Service 的静态 IP 地址只能在集群内访问。

除了“ClusterIP”，Service 还支持其他三种类型，分别是“**ExternalName**”“**LoadBalancer**”“**NodePort**”。不过前两种类型一般由云服务商提供，我们的实验环境用不到，所以接下来就重点看“NodePort”这个类型。

“NodePort”，**就会在节点原有基础上开启一个随机端口号**（多个pod共享），让外界也能够访问内部的服务。(除了集群内部使用的“80”端口，还多出了一个“30651”端口，这就是 Kubernetes 在节点上为 Service 创建的专用映射端口。)

Service 除了会对后端的 Pod 做负载均衡之外，还会在集群里的每个节点上创建一个独立的端口，用这个端口对外提供服务，这也正是“NodePort”这个名字的由来。

用 curl 访问 Kubernetes 集群的两个节点“192.168.10.210:30651”“192.168.10.220:30651”，就可以得到 Nginx Pod 的响应数据：

### Ingress

 Service 的功能和运行机制，它本质上就是一个由 kube-proxy 控制的四层负载均衡，在 TCP/IP 协议栈上转发流量。但在四层上的负载均衡功能还是太有限了。

Service 还有一个缺点，**它比较适合代理集群内部的服务**。如果想要把服务暴露到集群外部，就只能使用 NodePort 或者 LoadBalancer 这两种方式，而它们都缺乏足够的灵活性，难以管控。

Ingress可以做到**七层负载均衡，这个对象还应该承担更多的职责，也就是作为流量的总入口，统管集群的进出口数据**，“扇入”“扇出”流量（也就是我们常说的“南北向”），让外部用户能够安全、顺畅、便捷地访问内部服务。它基于 HTTP/HTTPS 协议定义路由规则。

Ingress 只是规则的集合，自身不具备流量管理能力，需要 Ingress Controller 应用 Ingress 规则才能真正发挥作用。

Ingress Controller，它的作用就相当于 Service 的 kube-proxy，能够读取、应用 Ingress 规则，处理、调度流量。

**不过 Ingress Controller 要做的事情太多，与上层业务联系太密切，所以 Kubernetes 把 Ingress Controller 的实现交给了社区**，任何人都可以开发 Ingress Controller，只要遵守 Ingress 规则就好。比如社区的 Kubernetes Ingress Controller和Nginx Ingress Controller

但随着 Ingress 在实践中的大量应用，很多用户发现这种用法会带来一些问题，比如：

- 由于某些原因，项目组需要引入不同的 Ingress Controller，但 Kubernetes 不允许这样做；

- Ingress 规则太多，都交给一个 Ingress Controller 处理会让它不堪重负；
- 多个 Ingress 对象没有很好的逻辑分组方式，管理和维护成本很高；
- 集群里有不同的租户，他们对 Ingress 的需求差异很大甚至有冲突，无法部署在同一个 Ingress Controller 上。

所以，Kubernetes 就又提出了一个 Ingress Class 的概念，让它插在 Ingress 和 Ingress Controller 中间，作为流量规则和控制器的协调人，解除了 Ingress 和 Ingress Controller 的强绑定关系。

Ingress Class 解耦了 Ingress 和 Ingress Controller，我们应当使用 Ingress Class 来管理 Ingress 资源。

因为 Ingress Controller 本身也是一个 Pod，想要向外提供服务还是要依赖于 Service 对象。所以你至少还要再为它定义一个 Service，使用 NodePort 或者 LoadBalancer 暴露端口，才能真正把集群的内外流量打通。这个工作就交给你课下自己去完成了。

Ingress 的路由规则是 HTTP 协议，所以就不能用 IP 地址的方式访问，必须要用域名、URI。



### pv

**作为存储的抽象，PV 实际上就是一些存储设备、文件系统**，比如 Ceph、GlusterFS、NFS，甚至是本地磁盘。

PersistentVolumeClaim，简称 PVC，就是用来向 Kubernetes 申请存储资源的。PVC 是给 Pod 使用的对象，它相当于是 Pod 的代理，代表 Pod 向系统申请 PV。一旦资源申请成功，Kubernetes 就会把 PV 和 PVC 关联在一起，这个动作叫做“**绑定**”（bind）。

但是，系统里的存储资源非常多，如果要 PVC 去直接遍历查找合适的 PV 也很麻烦，所以就要用到 StorageClass。

StorageClass 的作用有点像第 21 讲里的 IngressClass，它抽象了特定类型的存储系统（比如 Ceph、NFS），在 PVC 和 PV 之间充当“协调人”的角色，帮助 PVC 找到合适的 PV。也就是说它可以简化 Pod 挂载“虚拟盘”的过程，让 Pod 看不到 PV 的实现细节。

实际yaml： pod+pvc（定义sc）+pv

### pv+NFS

 CSI（Container Storage Interface）

- 可以编写 PV 手工定义 NFS 静态存储卷，要指定 NFS 服务器的 IP 地址和共享目录名。
- 使用 NFS 动态存储卷必须要部署相应的 Provisioner，在 YAML 里正确配置 NFS 服务器。
- 动态存储卷不需要手工定义 PV，而是要定义 StorageClass，由关联的 Provisioner 自动创建 PV 完成绑定。



### StatefulSet

对于“有状态应用”，多个实例之间可能存在依赖关系，比如 master/slave、active/passive，需要依次启动才能保证应用正常运行，外界的客户端也可能要使用固定的网络标识来访问实例，而且这些信息还必须要保证在 Pod 重启后不变。

和 DaemonSet 类似，StatefulSet 也可以看做是 Deployment 的一个特例。

StatefulSet有了启动的先后顺序，应用该怎么知道自己的身份，进而确定互相之间的依赖关系呢？

Kubernetes 给出的方法是**使用 hostname**，也就是每个 Pod 里的主机名。

在 Pod 里查看环境变量 $HOSTNAME 或者是执行命令 hostname，都可以得到这个 Pod 的名字 redis-sts-0。

有了这个唯一的名字，应用就可以自行决定依赖关系了，比如在这个 Redis 例子里，就可以让先启动的 0 号 Pod 是主实例，后启动的 1 号 Pod 是从实例。

Service 发现这些 Pod 不是一般的应用，而是有状态应用，需要有稳定的网络标识，所以就会为 Pod 再多创建出一个新的域名，格式是“**Pod 名. 服务名. 名字空间.svc.cluster.local**”。当然，这个域名也可以简写成“**Pod 名. 服务名**”。

StatefulSet 里的这两个 Pod 都有了各自的域名，也就是稳定的网络标识。那么接下来，外部的客户端只要知道了 StatefulSet 对象，就可以用固定的编号去访问某个具体的实例了，虽然 Pod 的 IP 地址可能会变，但这个有编号的域名由 Service 对象维护，是稳定不变的。

到这里，通过 StatefulSet 和 Service 的联合使用，Kubernetes 就解决了“有状态应用”的**依赖关系、启动顺序和网络标识**这三个问题。

- 要为 StatefulSet 里的 Pod 生成稳定的域名，需要定义 Service 对象，它的名字必须和 StatefulSet 里的 serviceName 一致。
- 访问 StatefulSet 应该使用每个 Pod 的单独域名，形式是“Pod 名. 服务名”，不应该使用 Service 的负载均衡功能。
- 在 StatefulSet 里可以用字段“volumeClaimTemplates”直接定义 PVC，让 Pod 实现数据持久化存储。



###  kubectl rollout 实现用户无感知的应用升级和降级

Kubernetes 不是把旧 Pod 全部销毁再一次性创建出新 Pod，而是在逐个地创建新 Pod，同时也在销毁旧 Pod，保证系统里始终有足够数量的 Pod 在运行，不会有“空窗期”中断服务。

新 Pod 数量增加的过程有点像是“滚雪球”，从零开始，越滚越大，所以这就是所谓的“**滚动更新**”（rolling update）。

**想要回退到上一个版本，就可以使用命令** **kubectl rollout undo****，也可以加上参数** **--to-revision** **回退到任意一个历史版本**。



### 容器资源配额

具体的申请方法很简单，**只要在 Pod 容器的描述部分添加一个新字段** **resources** **就可以了**，它就相当于申请资源的 Claim。

containers.resources，它下面有两个字段：

- “**requests**”，意思是容器要申请的资源，也就是说要求 Kubernetes 在创建 Pod 的时候必须分配这里列出的资源，否则容器就无法运行。
- “**limits**”，意思是容器使用资源的上限，不能超过设定值，否则就有可能被强制停止运行。

### 容器状态探针

Kubernetes 为检查应用状态定义了三种探针，它们分别对应容器不同的状态：

- **Startup**，启动探针，用来检查应用是否已经启动成功，适合那些有大量初始化工作要做，启动很慢的应用。
- **Liveness**，存活探针，用来检查应用是否正常运行，是否存在死锁、死循环。
- **Readiness**，就绪探针，用来检查应用是否可以接收流量，是否能够对外提供服务。

至于探测方式，Kubernetes 支持 3 种：Shell、TCP Socket、HTTP GET，它们也需要在探针里配置：

- **exec**，执行一个 Linux 命令，比如 ps、cat 等等，和 container 的 command 字段很类似。
- **tcpSocket**，使用 TCP 协议尝试连接容器的指定端口。
- **httpGet**，连接端口并发送 HTTP GET 请求。



### 用名字空间分隔系统资源

当多团队、多项目共用 Kubernetes 的时候，为了避免这些问题的出现，我们就需要**把集群给适当地“局部化”，为每一类用户创建出只属于它自己的“工作空间”**。

**名字空间的资源配额需要使用一个专门的 API 对象，叫做** **ResourceQuota****，简称是** **quota**，我们可以使用命令 kubectl create 创建一个它的样板文件：

在 ResourceQuota 里可以设置各类资源配额，字段非常多，我简单地归了一下类，你可以课后再去官方文档上查找详细信息：

- CPU 和内存配额，使用 request.*、limits.*，这是和容器资源限制是一样的。
- 存储容量配额，使 requests.storage 限制的是 PVC 的存储总量，也可以用 persistentvolumeclaims 限制 PVC 的个数。
- 核心对象配额，使用对象的名字（英语复数形式），比如 pods、configmaps、secrets、services。
- 其他 API 对象配额，使用 count/name.group 的形式，比如 count/jobs.batch、count/deployments.apps。

##### 默认资源配额

这个时候就要用到一个**很小但很有用的辅助对象了——** **LimitRange****，简称是** **limits****，它能为 API 对象添加默认的资源配额限制**。



- 名字空间是一个逻辑概念，没有实体，它的目标是为资源和对象划分出一个逻辑边界，避免冲突。
- ResourceQuota 对象可以为名字空间添加资源配额，限制全局的 CPU、内存和 API 对象数量。
- LimitRange 对象可以为容器或者 Pod 添加默认的资源配额，简化对象的创建工作。



### Metrics Server

Metrics Server 是一个专门用来收集 Kubernetes 核心资源指标（metrics）的工具，它定时从所有节点的 kubelet 里采集信息。

### HorizontalPodAutoscaler

它是专门用来自动伸缩 Pod 数量的对象，适用于 Deployment 和 StatefulSet，但不能用于 DaemonSet。

HorizontalPodAutoscaler 的能力完全基于 Metrics Server，它从 Metrics Server 获取当前应用的运行指标，主要是 CPU 使用率，再依据预定的策略增加或者减少 Pod 的数量。

### Prometheus

### 网络通信：CNI

针对 Docker 的网络缺陷，Kubernetes 提出了一个自己的网络模型“**IP-per-pod**”，能够很好地适应集群系统的网络需求，它有下面的这 4 点基本假设：

- 集群里的每个 Pod 都会有唯一的一个 IP 地址。
- Pod 里的所有容器共享这个 IP 地址。
- 集群里的所有 Pod 都属于同一个网段。
- Pod 直接可以基于 IP 地址直接访问另一个 Pod，不需要做麻烦的网络地址转换（NAT）。

CNI 为网络插件定义了一系列通用接口，开发者只要遵循这个规范就可以接入 Kubernetes，为 Pod 创建虚拟网卡、分配 IP 地址、设置路由规则，最后就能够实现“IP-per-pod”网络模型。

依据实现技术的不同，CNI 插件可以大致上分成“**Overlay**”“**Route**”和“**Underlay**”三种。

**Overlay** 的原意是“覆盖”，是指它构建了一个工作在真实底层网络之上的“逻辑网络”，把原始的 Pod 网络数据封包，再通过下层网络发送出去，到了目的地再拆包。因为这个特点，它对底层网络的要求低，适应性强，缺点就是有额外的传输成本，性能较低。

**Route** 也是在底层网络之上工作，但它没有封包和拆包，而是使用系统内置的路由功能来实现 Pod 跨主机通信。它的好处是性能高，不过对底层网络的依赖性比较强，如果底层不支持就没办法工作了。

**Underlay** 就是直接用底层网络来实现 CNI，也就是说 Pod 和宿主机都在一个网络里，Pod 和宿主机是平等的。它对底层的硬件和网络的依赖性是最强的，因而不够灵活，但性能最高。





pod生命周期

- pod创建过程

- 运行初始化容器（init container）过程

- 运行主容器（main container）

- - 容器启动后钩子（post start）、容器终止前钩子（pre stop）
  - 容器的存活性检测（liveness probe）、就绪性检测（readiness probe）

- pod终止过程
