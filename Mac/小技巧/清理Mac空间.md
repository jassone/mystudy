## 清理Mac空间

在“系统”中，各应用的缓存及日志文件可放心清理，找到对应的目录直接删除即可；而应用的其他文件，在磁盘空间不够时，大家可选择性清除数据文件。另外，卸载不常使用的应用也可以增加磁盘空间。

### 1、系统缓存及日志文件位置：
* 系统缓存保存在：~/Library/Caches
* 系统日志保存在：~/Library/Logs

### 2、应用缓存及日志文件位置：
App Store下载的应用：
缓存文件保存在：~/Library/Containers/com.xx.xx(应用名称)/Data/Library/Caches
日志文件保存在：~/Library/Containers/com.xx.xx(应用名称)/Data/Library/Logs

可以使用命令：“sudo du -sh * ”查看当前文件夹下各个文件和文件夹占用的空间大小，进而一步步找到占用磁盘空间较多的文件。

### 3、其他

*  ~/Library/Group Containers/UBF8T346G9.Office/  office报错信息
*  ~/Library/Containers/
*  ~/Library/Application Support/dinglive
*  ~/Library/Application Support/DingTalkMac
*  ~/Library/Application Support/google
*  /Users/bian/.cache    有时候有些程序把缓存放在这里