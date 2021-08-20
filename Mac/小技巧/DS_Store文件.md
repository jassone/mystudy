### .DS_Store 是什么
Mac 系统经常会自动在每个目录生成一个隐藏的 .DS_Store 文件。.DS_Store(英文全称 Desktop Services Store)是一种由苹果公司的Mac OS X操作系统所创造的隐藏文件，目的在于存贮目录的自定义属性，例如文件们的图标位置或者是背景色的选择。相当于 Windows 下的 desktop.ini。

> Mac下 以.开头的都是隐藏文件

### 删除 .DS_Store （git）
删除项目中的所有.DS_Store。这会跳过不在项目中的 .DS_Store

```
删除项目中的所有.DS_Store。这会跳过不在项目中的 .DS_Store
1.find . -name .DS_Store -print0 | xargs -0 git rm -f --ignore-unmatch
将 .DS_Store 加入到 .gitignore
2.echo .DS_Store >> ~/.gitignore
更新项目
3.git add --all
4.git commit -m '.DS_Store banished!'
```

如果只需要删除磁盘上的 .DS_Store，可以使用下面的命令来删除当前目录及其子目录下的所有.DS_Store 文件:

```
find . -name '*.DS_Store' -type f -delete
```

### 禁用或启用自动生成
禁止.DS_store生成：

```
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
```

恢复.DS_store生成：恢复.DS_store生成：

```
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
 
```