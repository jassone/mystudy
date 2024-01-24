## k8s中重新部署pod时经历了哪些

## 杀掉老的pod

在 Kubernetes（k8s）中，当重新部署一个Pod时（例如，升级一个Deployment），对旧的Pod的处理方式取决于Pod的`terminationGracePeriodSeconds`和容器中的`preStop`钩子。

Kubernetes 首先执行以下步骤：

1. 为旧的Pod（要被停止的Pod）发送`SIGTERM`信号。这个`SIGTERM`信号将被直接发送到容器运行的进程中。
2. 如果为容器定义了`preStop`钩子（lifecycle hook），在发送`SIGTERM`信号之后，Kubernetes 将执行`preStop`钩子。这个钩子可以是一个命令或者一个HTTP请求。这允许您在Pod被终止前执行一些操作，例如优雅地关闭服务或通知其他组件。这个钩子应在`SIGTERM`接收后尽快执行完成，但如果执行时间过长，它可能会超过指定的`terminationGracePeriodSeconds`。
3. 在`terminationGracePeriodSeconds`时间结束后（默认为30秒），Kubernetes 会给容器发送一个`SIGKILL`信号，并停止容器。这个信号无法被捕获或忽略，所以容器将立即终止。

为了优雅地关闭服务和处理连接，应用程序应该侦听并正确处理`SIGTERM`信号。这样，当 Kubernetes 重新部署一个Pod或缩放应用程序时，应用程序将有一定的宽限期来清理资源和关闭连接。