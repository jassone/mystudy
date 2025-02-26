## scp

scp 是 secure copy 的缩写, scp 是 linux 系统下基于 ssh 登陆进行安全的远程文件拷贝命令。

scp 是加密的，[rcp](https://www.runoob.com/linux/linux-comm-rcp.html) 是不加密的，scp 是 rcp 的加强版。



```
scp [可选参数] file_source file_target 
```



### 1、从本地复制到远程

```
scp local_file remote_username@remote_ip:remote_folder 
scp /home/space/music/1.mp3 root@www.runoob.com:/home/root/others/music 
```

### 2、从远程下载到本地

```
scp root@47.98.38.184:/root/.ssh/known_hosts  /Users/bian/aliy
```

