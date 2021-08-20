### xcrun: error: invalid active developer path (/Applications/Xcode.app/Contents/Developer)解决办法

mac下卸载了xcode，使用git等命令时就提示错误。invalid active path(Applications/Xcode.app/Contents/Developer),这种情况可以通过xcode-select --switch指定一个xcode安装路径，如果不想安装xcode,那么可以通过重置系统默认开发工具路径.

```
sudo xcode-select -r

xcode-select --switch /Library/Developer/CommandLineTools

xcode-select -p
```