## 各个io库对比

在Go语言中涉及 I/O 操作的内置库有很多种，比如： io 库， os 库， ioutil 库， bufio 库， bytes 库， strings 库等等。拥有这么多内置库是好事，但是具体到涉及 I/O 的场景我们应该选择哪个库呢？

## 一、 io.Reader/Writer

Go语言里使用 io.Reader 和 io.Writer 两个 interface 来抽象 I/O ，他们的定义如下。

```go
type Reader interface {
 Read(p []byte) (n int, err error)
}

type Writer interface {
 Write(p []byte) (n int, err error)
}
```

**io.Reader 接口代表一个可以从中读取字节流的实体，而 io.Writer 则代表一个可以向其写入字节流的实体。**

io.Reader/Writer 常用的几种实现

- net.Conn: 表示网络连接。

  ```go
  listener,err := net.Listen("tcp",addr)
  if err != nil {
    panic(err)
  }
  
  conn,err := listener.Accept()
  if err != nil {
    panic(err)
  }
  defer conn.Close()
  
  reader := bufio.NewReader(conn)
  var buf [1024]byte
  for {
  	n, err := reader.Read(buf[:])
  	if err == io.EOF {
  		break
  	}
  	if err != nil {
  		fmt.Println("read from client failed, err:", err)
  		break
  	}
  	recvStr := string(buf[:n])
  	fmt.Println("收到client发来的数据：", recvStr)
  }
  ```

- os.Stdin, os.Stdout, os.Stderr: 标准输入、输出和错误。

  ```go
  reader := bufio.NewReader(os.Stdin) //os.Stdin 代表标准输入[终端]
  for {
    //从终端读取一行用户输入，并准备发送给服务器
    line, err := reader.ReadString('\n') // 阻塞从键盘读
    if err != nil {
      log.Fatal(err)
    }
    line = strings.Trim(line, "\r\n")
  }
  ```

- os.File: 文件的流读取。

- strings.Reader: 字符串抽象成 io.Reader 的实现。

- bytes.Reader: []byte抽象成 io.Reader 的实现。

- bytes.Buffer: []byte抽象成 io.Reader 和 io.Writer 的实现。

- bufio.Reader/Writer: 抽带缓冲的流读取和写入（比如按行读写）。

除了这几种实现外常用的还有 ioutil 工具库包含了很多IO工具函数，编码相关的内置库 encoding/base64、encoding/binary 等也是通过 io.Reader 和 io.Writer 实现各自的编码功能的。

## 二、每种I/O库的使用场景

### 1、io库-按字节读取

io 库属于底层接口定义库，包含Reader和Writer两个接口，这两个接口被很多type实现，如FIle。

它的作用主要是定义个 I/O 的基本接口和个基本常量，并解释这些接口的功能。**在实际编写代码做 I/O 操作时，这个库一般只用来调用它的常量和接口定义**，比如用 io.EOF 判断是否已经读取完，用 io.Reader 做变量的类型声明。

```go
// 字节流读取完后，会返回io.EOF这个error
for {
 n, err := r.Read(buf)
 fmt.Println(n, err, buf[:n])
 if err == io.EOF {
  break
 }
}
```

### 2、os 库

os 库主要是处理操作系统操作的，它作为Go程序和操作系统交互的桥梁。创建文件、打开或者关闭文件、Socket等等这些操作和都是和操作系统挂钩的，所以都通过 os 库来执行。这个库经常和 ioutil ， bufio 等配合使用。

```go
//将打开文件时产生的error作为参数传递，如果文件不存在，则返回false
func IsExist(err error) bool
//打开文件产生的error，作为参数传递，如果是因为没有权限而产生错误，则返回true
func IsPermission(err error) bool
//返回当前的路径
func Getwd() (dir string, err error)
//根据不同权限创建目录
func Mkdir(name string, perm FileMode) error
//将文件进行更改名称
func Rename(oldpath, newpath string) error
//删除文件或者目录
func Remove(name string) error
//创建文件，默认权限是0666
func Create(name string) (file *File, err error)
//打开文件，是只读模式。
func Open(name string) (file *File, err error)
//打开文件，可以指定打开文件的模式
func OpenFile(name string, flag int, perm FileMode) (file *File, err error)
//读取文件的数据，注意，每次最多读取有len(b)个字符
func (f *File) Read(b []byte) (n int, err error)
//向文件中写入数据，byte切片
func (f *File) Write(b []byte) (n int, err error)
//向文件中写入数据，string
func (f *File) WriteString(s string) (ret int, err error)
//关闭打开文件的句柄
func (f *File) Close() error
```

### 3、ioutil库

ioutil 库是一个有工具包，它提供了很多使用的 IO 工具函数，例如 ReadAll、ReadFile、WriteFile、ReadDir。**唯一需要注意的是它们都是一次性读取和一次性写入**，所以使用时，**尤其是把数据从文件里一次性读到内存中时需要注意文件的大小**。

读出文件中的所有内容

```go
func readByFile() {
  data, err := ioutil.ReadFile( "./file/test.txt")
  if err != nil {
    log.Fatal("err:", err)
    return
  }
  fmt.Println("data", string(data)) 
}
```

将数据一次性写入文件

```go
func writeFile() {
  err := ioutil.WriteFile("./file/write_test.txt", []byte("hello world!"), 0644)
  if err != nil {
    panic(err)
    return
  }
}
```

```go
//将os.open()等返回的操作文件的句柄，作为参数传递，如果遇到io.EOF或者其他error将停止读取数据
func ReadAll(r io.Reader) ([]byte, error)
//从文件中读物数据，读取成功后的error返回的error为nil
func ReadFile(filename string) ([]byte, error)
//将数据写入文件
func WriteFile(filename string, data []byte, perm os.FileMode) error
```

### 4、bufio库-按行读取

bufio，**可以理解为在 io 库的基础上额外封装加了一个缓存层**，它提供了很多按行进行读写的函数，**从io库的按字节读写变为按行读写对写代码来说还是方便了不少。**

- ReadLine和ReadString方法：**buf.ReadLine()，buf.ReadString(" ")都是按行读，只不过ReadLine读出来的是[]byte，后者直接读出了string，最终他们底层调用的都是ReadSlice方法。**
- bufio VS ioutil 库：**bufio VS 和 ioutil 库都提供了读写文件的能力。它们之间唯一的区别是 bufio 有一个额外的缓存层。这个优势主要体现在读取大文件的时候。**

```go
//将os.open等打开的文件作为参数传入，从writer中读取数据
// 用的
func NewReader(rd io.Reader) *Reader

//从reader中读取数据，每次最多读取len(p)的数据量，如果遇到结尾，则返回error为io.EOF
func (b *Reader) Read(p []byte) (n int, err error)
//读取一行数据，以\n为分割
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
//将文件中的数据使用delim进行分割，对分割的数据以byte形式返回
func (b *Reader) ReadBytes(delim byte) (line []byte, err error)
//将文件中的数据使用delim进行分割，返回的数据以string形式返回
func (b *Reader) ReadString(delim byte) (line string, err error)

//创建写文件的句柄，将os.open等返回的*file进行传递
// 用的比较多
func NewWriter(w io.Writer) *Writer

//将数据写入缓冲，将写入的数据进行统计，如果写入的数据少于传递的参数，则返回错误
func (b *Writer) Write(p []byte) (nn int, err error)
//在文件中写入字符串，写入的数据进行统计，如果写入的数据少于传递的参数，则返回一个错误
func (b *Writer) WriteString(s string) (int, error)
//将缓存的数据写入文件
func (b *Writer) Flush() error
//传入写的句柄和读的句柄，并返回一个写和都都具有的句柄，并将读写分配到读写句柄中
func NewReadWriter(r *Reader, w *Writer) *ReadWriter
```

### 5、bytes 和 strings 库

bytes 和 strings 库里的 bytes.Reader 和string.Reader，它们都实现了 io.Reader 接口，也都提供了NewReader方法用来从 []byte 或者 string 类型的变量直接构建出相应的Reader实现。**所以数据源不一定是文件，也可以是网络数据等。**

```go
r := strings.NewReader("abcde")
// 或者是 bytes.NewReader([]byte("abcde"))
buf := make([]byte, 4)
for {
 n, err := r.Read(buf)
 fmt.Println(n, err, buf[:n])
 if err == io.EOF {
  break
 }
}
```

另一个区别是 bytes 库有Buffer的功能，而 strings 库则没有。

```go
var buf bytes.Buffer
fmt.Fprintf(&buf, "Size: %d MB.", 85)
s := buf.String()) // s == "Size: 85 MB."
```

## 三、总结

关于 io.Reader 和 io.Writer 接口，可以简单理解为读源和写源。也就是说，只要实现了 Reader 中的 Read 方法，这个东西就可以作为读源，里面可以包含数据，被我们读取。 Writer 也是如此。

