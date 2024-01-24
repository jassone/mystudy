## main

 ```
 python -i 1.py // 进入解释器中，交互方式模式
 ```

强类型，需要做类型转换，必须显示改变



- 常量：None
- 空集："",(),[],{},set(),range(0)

 ### 表示假值

None，False，0, 0.00， 空的序列，空dict



### 格式化输出

- 百分号%
- format() 函数
- f-strings  目前用的多 



### 其他

with open(文件) as 文件描述符  打开，并会自动关闭文件

### 导入

```
import dir1.dir2.文件名
如果在同一个目录，则可以省略目录
```

### 虚拟环境的使用

- 解决多个模块依赖的问题
- 一次性安装多个指定版本的模块
- 避免对默认环境造成污染

```
python -m venv myvenv

// venv 虚拟环境模块
// myvenv 保存虚拟环境的文件夹

pip3 freeze > requirements.txt // 将当前安装的包及其版本保存到文件夹中
source myvenv/bin/activate // 激活虚拟环境
pip3 install -f requirements.txt // 在虚拟环境中导入指定的包
deactivate // 离开虚拟环境
```





## 其他

命令行：help(list)  : 查看list的相关文档
