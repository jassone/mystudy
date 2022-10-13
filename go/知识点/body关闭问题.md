## body关闭问题

## 一、为什么要关闭resp.body

> Body io.ReadCloser
>
> The http Client and Transport guarantee that Body is always non-nil, even on 
> responses without a body or responses with a zero-length body. It is the caller's 
> responsibility to close Body. The default HTTP client's Transport does not attempt to 
> reuse HTTP/1.0 or HTTP/1.1 TCP connections ("keep-alive") unless the Body is read to 
> completion and is closed.
>
> http客户端(Client)和传输(Transport)保证响应体总是非空的，即使响应没有响应体或0长响应
> 体。关闭响应体是调用者的责任。默认http客户端传输(Transport)不会尝试复用keep-alive的
> http/1.0、http/1.1连接，除非请求体已被完全读出而且被关闭了。

以上是http包文档说明。但是为什么body需要被关闭呢，不关闭会如何？那就读源码呗。

### 1、body是什么

要了解body，首先要了解http事务是如何处理的。http事务是交由底层的Transport处理的。

第一步是从连接池获取一个连接，这个连接的功能由3个goroutine协同实现，一个主goroutine，一个readLoop，一个writeLoop，后两个goroutine生命周期和连接一致。虽说readLoop和writeLoop名字叫循环（也确实是for循环），但实际上一次循环就完整处理一个http事务，循环本身仅仅是为了连接复用，所以为了便于理解其逻辑可以忽略它的循环结构。

接下来三个goroutine协同完成http事务：

- 主goroutine将request同时发给readLoop和writeLoop。
- writeLoop发送request，然后将状态（error）发送给主goroutine和readLoop。
- readLoop解析头部response，然后将状态（error）和response发送给主goroutine。
- 主goroutine返回用户代码，readLoop等待body读取完成。
- readLoop回收连接。

了解http事务的处理流程，然后我们回过头来看看神秘的body到底是什么

```go
//源码版本1.8.3
// src/net/http/transfer.go:405 body解析方法
func readTransfer(msg interface{}, r *bufio.Reader) (err error)

// src/net/http/transfer.go:485 解析chunked
t.Body = &body{src: internal.NewChunkedReader(r), hdr: msg, r: r, closing: t.Close}

// src/net/http/transfer.go:490 产生eof
t.Body = &body{src: io.LimitReader(r, realLength), closing: t.Close}

// src/net/http/transport.go:1560 发送eof信号
body := &bodyEOFSignal{

// src/net/http/transport.go:1583 gzip解码
resp.Body = &gzipReader{body: body}
```

body实际上是一个嵌套了多层的net.TCPConn：

（1）bufio.Reader，这层尝试将多次小的读操作替换为一次大的读操作，减少系统调用的次数，提高性能；
（2）io.LimitedReader，tcp连接在读取完body后不会关闭，继续读会导致阻塞，所以需要LimitedReader在body读完后发出eof终止读取；
（3）chunkedReader，解析chunked格式编码（如果不是chunked略过）；
（4）bodyEOFSignal，在读到eof，或者是提前关闭body时会对readLoop发出回收连接的通知；
（5）gzipReader，解析gzip压缩（如果不是gizp压缩略过）；

**从上面可以看出如果body既没有被完全读取，也没有被关闭，那么这次http事务就没有完成，除非连接因超时终止了，否则相关资源无法被回收。**

如果请求头或响应头指明Connection: close呢？还是无法回收，因为close表示在http事务完成后断开连接，而事务尚未完成自然不会断开，更不会回收。

从实现上看只要body被读完，连接就能被回收，只有需要抛弃body时才需要close，似乎不关闭也可以。但那些正常情况能读完的body，即第一种情况，在出现错误时就不会被读完，即转为第二种情况。而分情况处理则增加了维护者的心智负担，所以始终close body是最佳选择。

## 二 resp.body 关闭方法

如果我们需要做大量的并发http请求，所以经常需要打开和关闭大量http连接。 如果没有正确处理http连接， 很容易导致内存的泄漏。（这里和http的连接复用区分开，http连接复用实际上指的是多个http连接复用下层tcp连接，也就是俗称的长连接，具体参考：http连接复用）

### 1、错误做法

错误实例1

```go
resp, err := http.Get("https://api.ipify.org?format=json")
defer resp.Body.Close() //not ok
if err != nil {
    fmt.Println(err)
    return
}
body, err := ioutil.ReadAll(resp.Body)
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(string(body))
```
这么写是不对的， 如果请求失败的话， resp就可能为nil值， 此时就会panic。

错误示例2

```go
resp, err := http.Get("https://api.ipify.org?format=json")
if err != nil {
    fmt.Println(err)
    return
}
defer resp.Body.Close()//ok, most of the
time :-)
body, err := ioutil.ReadAll(resp.Body)
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(string(body)) 
```
大多数情况下，当你的http请求失败， resp变量会为nil， err变量会为非nil值。 那么上面这种写法是没有错的。

但是， 当你得到的是一个重定向错误时， resp和err变量都会变为非nil值。 这意味着在第一个if判断 err != nil 的时候就返回了，此时走不到defer语句，无法关闭resp.Body，但是因为body里面非空，占用了一个连接，就会导致内存的泄漏。

### 2、正确做法

- 在结果处理的时候， 判断resp变量如果为非nil值， 就调用close。
- 另一个方式是， 不论成功失败，退出前都使用defer 调用close去关闭response.Body。

```go
resp, err := http.Get("https://api.ipify.org?format=json")
if resp != nil {
    defer resp.Body.Close()
}
if err != nil {
    fmt.Println(err)
    return
}

body, err := ioutil.ReadAll(resp.Body)
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(string(body)) 
```
原来旧版本的resp.Body.Close()实现中， 会替你读取和丢弃body中的残留数据。 这样可以确保这个http连接可以被其他请求重用， 比如在一些使能了http长连接的场景中。

但是 最新的Body.Close函数实现是不一样的。需要自己去读取和丢弃body数据。 如果不手动这么做， http会被关闭，而无法被重用。

如果你需要重用http连接， 那你需要在代码底部添加如下代码：

```go
_, err = io.Copy(ioutil.Discard, resp.Body)  
```

假设不需要立刻读取整个响应body， 比如在流式处理json响应的时候，可以用下面实现：

```go
json.NewDecoder(resp.Body).Decode(&data)   
```

## 三、扩展

### 1、如果不close，查看内存泄漏数量

##### a) 执行ioutil.ReadAll的情况(同域名)

```go
num := 2
for index := 0; index < num; index++ {
  resp, _ := http.Get("https://www.baidu.com")
  _, _ = ioutil.ReadAll(resp.Body)
}
fmt.Printf("此时goroutine个数= %d\n", runtime.NumGoroutine())
```

 上面这个在不执行resp.Body.Close()的情况下，泄漏了吗？如果泄漏，泄漏了多少个goroutine?

不进行resp.Body.Close()，泄漏是一定的。但是泄漏的goroutine个数就让我迷糊了。由于执行了2遍，每次泄漏一个读和写goroutine，就是4个goroutine，加上main函数本身也是一个goroutine，所以答案是5.
然而执行程序，发现答案是3，出入有点大，为什么呢？

- 虽然执行了 2 次循环，而且每次都没有执行 Body.Close() ,就是因为执行了ioutil.ReadAll()把内容都读出来了，连接得以复用，因此只泄漏了一个读goroutine和一个写goroutine，最后加上main goroutine，所以答案就是3个goroutine。
- 从另外一个角度说，正常情况下我们的代码都会执行 ioutil.ReadAll()，但如果此时忘了 resp.Body.Close()，确实会导致泄漏。但如果你调用的域名一直是同一个的话，那么只会泄漏一个 读goroutine 和一个写goroutine，这就是为什么代码明明不规范但却看不到明显内存泄漏的原因。

##### b) 未执行ioutil.ReadAll的情况(同域名)

此时打印出来的是5，原因是：每次泄漏一个读和写goroutine，就是4个goroutine，加上main函数本身也是一个goroutine，总共是5。此时连接不能复用。

##### c) 执行ioutil.ReadAll的情况(不同域名)

```go
num := 2
for index := 0; index < num; index++ {
  resp, _ := http.Get("https://www.baidu.com")
  _, _ = ioutil.ReadAll(resp.Body)
  if resp !=nil {
  }

  resp2, _ := http.Get("https://wenku.baidu.com/")
  _, _ = ioutil.ReadAll(resp.Body)
  if resp2 !=nil {
  }

}
fmt.Printf("此时goroutine个数= %d\n", runtime.NumGoroutine())
```

此时打印是7(1次循环是5)，原因是：？？todo

##### d) 未执行ioutil.ReadAll的情况(不同域名）

此时打印是9(1次循环是5)，原因是：见上面b的情况。