## main

### 1. 服务都正常起了，端口绑定也正常，访问时，报拒绝访问

docker run -d -p 8088:80  。。。

可能容器内映射没有搞好，排查如下：

- 进入容器：docker exec -it <container_id_or_name> /bin/bash

-  查看所有正在监听的端口  netstat -antp

  （或者仅查看 TCP 连接  netstat -antp | grep tcp，）  结果如下

  ```
  / # netstat -antp
  Active Internet connections (servers and established)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
  tcp        0      0 172.17.0.8:34310        172.17.0.1:19121        ESTABLISHED 1/lanyan-studio-go
  tcp        0      0 :::8083                 :::*                    LISTEN      1/lanyan-studio-go
  ```

  果然内部映射到8083端口了