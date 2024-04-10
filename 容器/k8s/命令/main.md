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

# 设置当前命名空间
kubectl config set-context --current --namespace={{namespace_name}}

# 查看某个命名空间下的pods
kubectl -n production get pods

# 查看默认命名空间下 pod 的详细信息
$ kubectl get pods -o wide 

# 实时监控 pod 的状态
$ kubectl get pod -w

## 进入容器
kubectl -n testing exec -it video-service-7888d9c4d8-kqlqn -- sh
kubectl -n production exec -it school-mina-service-654c8d67c7-s6sxm -- sh

# 将容器内某个文件下载到本地
kubectl cp production/school-mina-service-654c8d67c7-s6sxm:/storage/logs/logs.log  ./logs.log
tar: removing leading '/' from member names

## 本地文件上传到pod
kubectl cp ./logs.log  production/school-mina-service-654c8d67c7-s6sxm:/

## 删除pod
kubectl delete pod final-video-service-56db7df7b9-c6nht -n production

// 清理不需要的pod
kubectl -n production  get pods | grep Evicted |awk '{print$1}'|xargs kubectl -n production delete pods
```

