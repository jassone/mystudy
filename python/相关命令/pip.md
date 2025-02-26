## Pip命令

## 一、基本命令

``` pip list 
pip list  //查看安装的依赖
pip freeze // 可以查看已经安装的包及版本信息

pip uninstall torch // 卸载依赖

pip install -r requirements.txt  // 批量安装扩展
//protobuf
//transformers>=4.30.2

pip install transformers==4.30.2 // 如果有其他版本，则会替换

pip install dashscope // 
pip install dashscope --upgrade // 更新


python -m pip install --upgrade pip
pip install --upgrade openai // 升级


export PIP_DEFAULT_TIMEOUT=100  修改超时时间




python3 -m site // 查看环境路径
```

### 1、另外一种安装

- https://pypi.org/ 搜索某个扩展类库（https://blog.csdn.net/yuan2019035055/article/details/128709583），但是不能自动下载所

## 二、源修改

```
• 临时加速
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple package_name
pip3 install -r requirements.txt -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com

sudo pip3 install torch==2.1.2 --index-url=https://pypi.org/simple  切换源

• 永久加速
cat ~/pip.conf
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```

- 设置代理源： https://zhuanlan.zhihu.com/p/345161094
- pip配置国内镜像源 https://blog.51cto.com/u_15060510/3871395
- 脚本一键修改：https://gitee.com/shenzhiwuyan/updata_pip_Domestic-mirrors
- 【Python】怎么在pip下载的时候设置镜像？（常见的清华镜像、阿里云镜像以及中科大镜像）https://blog.csdn.net/wzk4869/article/details/130446850



## 三、下载慢

### 1、 HuggingFace

- https://zhuanlan.zhihu.com/p/647843635?utm_id=0  

  https://aliendao.cn/#/   用下载器 [model_donwload.py](https://e.aliendao.cn/model_download.py) 下载。已验证OK。

### 2、手动下载

```
# Download the package
wget https://files.pythonhosted.org/packages/{{path_to_downloaded_file}}

# Install the downloaded package
sudo pip3 install {{downloaded_file_name}}
```



