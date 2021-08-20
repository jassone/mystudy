## go build
命令主要用于编译代码。在包的编译过程中，若有必要，会同时编译与之相关联的包。


* go build 1.go  ，在当前目录下生产一个可执行文件

* go build -o hello2  hello.go   -o 指定可执行文件名称

* 如果当前目录下只有一个go文件，可省略后面的go文件名

* go build hello   则编译/Users/bian/webdata/go/src/hello下的文件

### 参数列表

* -a
     完全编译，不理会-i产生的.a文件(文件会比不带-a的编译出来要大？)
* -race
     同时检测数据竞争状态，只支持 linux/amd64, freebsd/amd64, darwin/amd64 和 windows/amd64.
 