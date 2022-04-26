## php多线程
php 本身不支持多线程，不过有些扩展可以帮你做到。

可以使用php的命令行模式运行，只不过这不是多线程，而是多进程。 ？？？

todo***

extension = "pthreads.so"

## 一、方式1-shell_exec函数
在PHP里使用shell_exec的函数，以shell的方式，启动一个独立的PHP脚本执行。
这种方式，其实相当于在Web服务器处理过程中，独立起了一个shell进程处理你的任务。这里，需要特别注意的是shell_exec的服务器安全，注意校验参数，小心避免被带入shell命令中。这个是比较容易实现的方式。

## 二、方式2-原生的pthread（多线程）
使用PHP实现一个Server，监听一个端口，为Web端提供服务。这里的实现方式有很多，通常要配合扩展，例如原生的pthread（多线程），开源扩展swoole等等。
 