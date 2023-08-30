## API



todo

https://zhuanlan.zhihu.com/p/524775243.  高质量



https://www.jianshu.com/p/73f3b22cded5

https://www.cnblogs.com/wuchangblog/p/16593137.html

https://blog.csdn.net/qq_35745940/article/details/121062767

https://www.cnblogs.com/yjf512/p/11174335.html

https://blog.csdn.net/xili2532/article/details/104543205

https://www.imooc.com/article/278725/

https://k8s.mybatis.io/v1.24 具体文档

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/。官方

https://blog.csdn.net/m0_46440313/article/details/124302783  有go中间件

https://github.com/kubernetes/client-go 官方维护的

https://github.com/ericchiang/k8s 社区维护的

https://blog.csdn.net/chenxy02/article/details/124254680 有go中间件

https://blog.csdn.net/kenkao/article/details/85626893



综合教程。https://www.apiref.com/kubernetes-zh/index.html



教程 http://b.jtthink.com/read.php?tid=596



官网定义 https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/



 自定义API资源类型(CRD)



一、namespaced resources

所谓的namespaced resources,就是这个resource是从属于某个namespace的, 比如pod, deployment, service都属于namespaced resource.

http://localhost:8080/api/v1/namespaces/default/pods/test-pod

可以看出, 该restful api的组织形式是:

api api版本　 namespaces 所属的namespace 资源种类 所请求的资源名称

api v1 namespaces default pods test-pod

二、non-namespaced resources

http://localhost:8080/apis/rbac.authorization.k8s.io/v1/clusterroles/test-clusterrole

这里可以观察到它clusterrole与pod不同, apis表示这是一个非核心api. rbac.authorization.k8s.io指代的是api-group, 另外它没有namespaces字段, 其他与namespaced resources类似.不再赘述.

三、non-resource url

这类资源和pod, clusterrole都不同. 例如

http://localhost:8080/healthz/etcd

这就是用来确认etcd服务是不是健康的.它不属于任何namespace,也不属于任何api版本.

总结, k8s的REST API的设计结构为:

[api/apis]/api-group/api-version/namespaces/namespace-name/resource-kind/resource-name

apis/rbac.authorization.k8s.io/v1/namespaces/default/roles/test-role