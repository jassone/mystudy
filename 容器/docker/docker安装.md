## docker安装

以Ubuntu为例，安装Docker Engine。

```sh
sudo apt install [-y] docker.io #还可以使用 -y 参数来避免确认，实现自动化操作

sudo service docker start         #启动docker服务
sudo usermod -aG docker ${USER}   #当前用户加入docker组
# 因为操作 Docker 必须要有 root 权限，而直接使用 root 用户不够安全，加入 Docker 用户组是一个比较好的选择，这也是 Docker 官方推荐的做法。当然，如果只是为了图省事，你也可以直接切换到 root 用户来操作 Docker。

exit # 重新登陆
```



## 相关文档

- Ubuntu安装 https://cloud.tencent.com/developer/article/2322853?areaId=106005
