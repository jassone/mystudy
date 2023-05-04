docker目前用的技术是containerd和runc。



pid namespace：用于隔离进程 ID。 net namespace：隔离网络接口，在虚拟的 net namespace 内用户可以拥有自己独立的 IP、路由、端口等。 mnt namespace：文件系统挂载点隔离。 ipc namespace：信号量,消息队列和共享内存的隔离。 uts namespace：主机名和域名的隔离。



普通的进程 + namespace（一重枷锁，能看到什么进程） +  cgroup（二重枷锁，能用多少资源，内存/磁盘。cpu等） +  chroot（三重枷锁，能看到什么文件）= 特殊的进程  = 容器

执行docker run -it ubuntu:18.04 sh后，在容器中执行ps，发现sh的pid为1，在宿主机上执行ps auxf | grep sh,发现是能看到容器内的这个sh进程的，但pid不同。此时在容器内执行apt update，宿主机上ps同样能看到，而且是sh的子进程。说明用户运行在容器里的应用进程，跟宿主机上的其他进程一样，都由宿主机操作系统统一管理，只不过通过namespace对这些进程做了特殊的隔离