## main

```
## 查看
# 查看命名空间
kubectl get ns / namespace   	

# 查看默认命名空间的 pod
$ kubectl get pods|pod|po

# 查看所有命名空间的 pod
$ kubectl get pods|pod -A
$ kubectl get pods|pod|po -n 命名空间名称

# 查看默认命名空间下 pod 的详细信息
$ kubectl get pods -o wide 

# 实时监控 pod 的状态
$ kubectl get pod -w


## 进入容器
kubectl -n testing exec -it video-service-7888d9c4d8-kqlqn -- sh


```

