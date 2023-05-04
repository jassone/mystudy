## bash/dash/sh的区别

## 一、sh

UNIX 最初使用，且在每种 UNIX 上都可以使用。在 shell 编程方面相当优秀，但在处理与用户的交互方面做得不如其他几种shell。

##  二、C shell (csh)

csh, the C shell, is a command interpreter with a syntax similar to the C programming language.一个语法上接近于C语言的shell。

## 三、 Korn shell (ksh)

完全向上兼容 Bourne shell 并包含了 C shell 的很多特性。

## 四、bash

因为Linux 操作系统缺省的 shell。即 bash 是 Bourne shell 的扩展，与 Bourne shell 完全向后兼容。在 Bourne shell 的基础上增加、增强了很多特性。可以提供如命令补全、命令编辑和命令历史表等功能。包含了很多 C shell 和 Korn shell 中的优点，有灵活和强大的编程接口，同时又有很友好的用户界面。

## 五、dash

原来bash是GNU/Linux 操作系统中的 /bin/sh 的符号连接，但由于bash过于复杂，有人把 bash 从 NetBSD 移植到 Linux 并更名为 dash，且/bin/sh符号连接到dash。Dash Shell 比 Bash Shell 小的多（ubuntu16.04上，bash大概1M，dash只有150K），符合POSIX标准。Ubuntu 6.10开始默认是Dash。

## 六、其他

### 1、查看当前系统支持的shell类型

```sh
$ cat /etc/shells
/bin/sh
/bin/dash
/bin/bash
/bin/rbash
```

### 2、命令比对

```sh
$ ll /bin/sh
lrwxrwxrwx 1 root root 4 3月   2  2016 /bin/sh -> dash*
$ ll /bin/dash
-rwxr-xr-x 1 root root 153960 3月   2  2016 /bin/dash*
$ ll /bin/bash
-rwxr-xr-x 1 root root 933936 6月  24  2019 /bin/bash*
$ ll /bin/rbash
lrwxrwxrwx 1 root root 4 6月  24  2019 /bin/rbash -> bash*
```

 