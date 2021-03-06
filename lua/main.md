## lua

lua设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。Lua由标准C编写而成，几乎在所有操作系统和平台上都可以编译，运行。Lua并没有提供强大的库，这是由它的定位决定的。**所以Lua不适合作为开发独立应用程序的语言**。

Lua脚本可以很容易的被C/C++ 代码调用，也可以反过来调用C/C++的函数，这使得Lua在应用程序中可以被广泛应用。不仅仅作为扩展脚本，也可以作为普通的配置文件，代替XML,ini等文件格式，并且更容易理解和维护。 Lua由标准C编写而成，代码简洁优美，几乎在所有操作系统和平台上都可以编译，运行。 一个完整的Lua解释器不过200k，在目前所有脚本引擎中，Lua的速度是最快的。这一切都决定了Lua是作为嵌入式脚本的最佳选择。

## 一、基础知识
### 1、Lua 特性
* 轻量级: 它用标准C语言编写并以源代码形式开放，编译后仅仅一百余K，可以很方便的嵌入别的程序里。
* 可扩展: Lua提供了非常易于使用的扩展接口和机制：由宿主语言(通常是C或C++)提供这些功能，Lua可以使用它们，就像是本来就内置的功能一样。
* 支持面向过程(procedure-oriented)编程和函数式编程(functional programming)；
* 自动内存管理；只提供了一种通用类型的表（table），用它可以实现数组，哈希表，集合，对象；
* 语言内置模式匹配；闭包(closure)；函数也可以看做一个值；提供多线程（协同进程，并非操作系统所支持的线程）支持；
* 通过闭包和table可以很方便地支持面向对象编程所需要的一些关键机制，比如数据抽象，虚函数，继承和重载等。
 
## 二、应用
* 游戏开发
* 独立应用脚本
* **Web 应用脚本，比如redis，nginx等**
* 扩展和数据库插件如：MySQL Proxy 和 MySQL WorkBench
* 安全系统，如入侵检测系统
 
## 三、相关wiki
* 官方：http://www.lua.org/
* 基础教程： https://www.runoob.com/lua/lua-tutorial.html

