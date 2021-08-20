 

## Launchctl 的介绍
Launchctl 是 Mac 系统自带的定时任务工具，与 crontab 功能类似。

## Launchctl 的使用
### 每 30 秒钟以定时任务运行

以简单的例子展示如何使用 Launchctl，比如现有一个位于 /Users/rundouble/Desktop 目录的 run.sh 脚本，脚本内容如下：

```
#!/bin/sh
echo `date`
```
并执行 chmod a+x run.sh 获取执行权限。

期望每 30 秒钟以定时任务运行上面的脚本。

首先，切换到管理员权限：

```
sudo su
```
进入 /Library/LaunchDaemons 目录下：

```
cd /Library/LaunchDaemons
```

创建 com.iamwr.launchctl.demo.agent.plist 定时任务描述文件：

```
vim com.iamwr.launchctl.demo.agent.plist
```
输入以下内容：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.iamwr.launchctl.demo.agent</string>
    <key>Program</key>
    <string>/Users/rundouble/Desktop/run.sh</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/rundouble/Desktop/run.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>30</integer>
    <key>StandardOutPath</key>
    <string>/Users/rundouble/Desktop/run.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/rundouble/Desktop/run.err</string>
</dict>
</plist>
```
##### .plist 文件标签说明：

* Label：定时任务的名称，这里 Label 的值是 com.iamwr.launchctl.demo.agent，要全局唯一，可通过：launchctl list 查看已有的定时任务

* Program：是要运行的脚本的名称，这里是 /Users/rundouble/Desktop/run.sh

* ProgramArguments： 脚本运行的参数，第一个是运行的命令，其余是运行需要的参数，这里是 /Users/rundouble/Desktop/run.sh，由于运行这个脚本不需要其他参数，所有只有这一个命令

* RunAtLoad：表示加载定时任务即开始执行脚本

* StartInterval： 定时任务的周期，单位为秒，这里 30 代表每 30s 执行一次脚本

* StandardOutPath： 标准输出日志的路径

* StandardErrorPath： 标准错误输出日志的路径

加载该定时任务：

```
launchctl load -w com.iamwr.launchctl.demo.agent.plist
```
-w 参数会将 .plist 文件中无意义的键值过滤掉，建议加上。

这时，定时任务已加载，由于加上了：

```
<key>RunAtLoad</key>
    <true/>
```
所以该定时任务成功加载后就开始执行，可以在 /Users/demo/run.log 看到每 30s 打印当前时间。

### 每天固定时间执行脚本
比如每天晚上 22:00 执行脚本，那么我们需要对 .plist 文件进行如下修改：

删除：
```
<key>StartInterval</key>
    <integer>30</integer>
```
添加：
```<key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>22</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
```
新的 .plist 文件内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.iamwr.launchctl.demo.agent</string>
    <key>Program</key>
    <string>/Users/rundouble/Desktop/run.sh</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/rundouble/Desktop/run.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>22</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/Users/rundouble/Desktop/run.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/rundouble/Desktop/run.err</string>
</dict>
</plist>
```
StartInterval 标签代表每多少秒执行，StartCalendarInterval 标签代表指定时间点执行，所以这两个标签在同一个 .plist 文件中只能存在一个。StartCalendarInterval 的 key 有：


| Key		| Type |  Values|
| --- | --- | --- |
| Month | Month | Month of year (1..12, 1 being January) |
| Day | Integer | Day of month (1..31) |
|Weekday  | Integer |  Day of week (0..7, 0 and 7 being Sunday)|
| Hour | Integer | Hour of day (0..23) |
| Minute | Integer | Minute of hour (0..59) |

 
同样使用 
```
launchctl load -w com.iamwr.launchctl.demo.agent.plist 
```
命令可使新的定时任务运行起来，在每天 22:00 时运行。

### Launchctl 细节
.plist 文件存放位置
上述展示中，.plist 描述文件是存放在 /Library/LaunchDaemons 目录下，除了这个目录还可以存在其他位置：

 
| Type		 |Location  |  Run on behalf of|
| --- | --- | --- |
|  User Agents|  ~/Library/LaunchAgents| Currently logged in user |
| Global Agents | /Library/LaunchAgents |  Currently logged in user|
|Global Daemons  | /Library/LaunchDaemons |  root or the user specified with the key UserName|
| System Agents | /System/Library/LaunchAgents | Currently logged in user |
| System Daemons | /System/Library/LaunchDaemons |  root or the user specified with the key UserName|
 
存放 在 LaunchAgents 代表用户登录后启动任务；存放在 LaunchDaemons 代表用户登录前就启动任务。一般将自定义的 .plist 存放在 /Library/LaunchDaemons 目录下。

### Launchctl 相关命令
查看已有的任务：

* launchctl list

查看指定的任务 xxx 是否存在（加载）：

* launchctl list | grep xxx

加载指定的任务 xxx：

* launchctl load -w xxx

或

* launchctl load xxx

删除任务:

* launchctl unload -w xxx

* launchctl load xxx

开始任务：

* launchctl start xxx

停止任务：

* launchctl stop xxx

##### 注意：

* 如果任务修改了，必须先 unload，然后重新执行 load

* start 可以测试任务，这个命令是立即执行，不管时间到了没有

* 执行 start 和 unload 命令前，任务必须先执行过 load，否则报错

* .plist 中不要添加 KeepAlive 标签，否则无条件 10s 定时执行
 