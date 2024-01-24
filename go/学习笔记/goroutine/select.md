## select

## 一、简介
**go select思想来源于网络IO模型中的select，本质上也是IO多路复用，只不过这里的IO是基于channel而不是基于网络。用于处理异步IO操作。**

**select 就是监听 IO 操作，当 IO 操作发生时，触发相应的动作每个case语句里必须是一个IO操作，确切的说，应该是一个面向channel的IO操作。**

> 注：Go 语言的 select 语句借鉴自 Unix 的 select() 函数，在 Unix 中，可以通过调用 select() 函数来监控一系列的文件句柄，一旦其中一个文件句柄发生了 IO 动作，该 select() 调用就会被返回（C 语言中就是这么做的），后来该机制也被用于实现高并发的 Socket 服务器程序。Go 语言直接在语言级别支持 select关键字，用于处理并发编程中通道之间异步 IO 通信问题。

## 二、特性

**select语句只能用于信道的读写操作(即每个case都必须是一个信道，都是IO操作)**

### 1、case条件选择

* 可处理一个或多个channel的发送/接收操作。
* **对于没有case的select{}或者没有default，会一直等待，可用于阻塞main函数。**

```go
select {
    case <-ch1:
        // 如果从 ch1 信道成功接收数据，则执行该分支代码
    case ch2 <- 1:
        // 如果成功向 ch2 信道成功发送数据，则执行该分支代码
    default:
        // 如果上面都没有成功，则进入 default 分支处理流程
}
```

##### 非阻塞情况
* case条件是并发执行的。select会选择先操作成功的那个case条件去执行。
* **如果多个同时返回，则随机选择一个执行，此时将无法保证执行顺序。**
* **通道被关闭了，会一直读取到零值**

##### 阻塞情况
* 对于阻塞的case语句会直到其中有信道可以操作
* 如果有多个信道可操作，会随机选择其中一个 case 执行

### 2、其他知识点
* 对于空的select{}，一直阻塞，有时候可以替代waitgroup的wait操作，更简洁。
* 对于for中的select{}, 也有可能会引起cpu占用过高的问题。
## 三、典型用法
### 1、超时判断

## 四、相关问题

### 1、如果保证ch1优先ch2执行

k8s源码中有处理

```go
func (tc *NoExecuteTaintManager) worker(worker int, done func(), stopCh <-chan struct{}) {
  defer done()
  for {
      select {
      case <-stopCh:
          return
      case nodeUpdate := <-tc.nodeUpdateChannels[worker]:
          tc.handleNodeUpdate(nodeUpdate)
          tc.nodeUpdateQueue.Done(nodeUpdate)
      case podUpdate := <-tc.podUpdateChannels[worker]:
          // If we found a Pod update we need to empty Node queue first.
      priority:
          for {
              select {
              case nodeUpdate := <-tc.nodeUpdateChannels[worker]:
                  tc.handleNodeUpdate(nodeUpdate)
                  tc.nodeUpdateQueue.Done(nodeUpdate)
              default:
                  break priority
              }
          }
          // After Node queue is emptied we process podUpdate.
          tc.handlePodUpdate(podUpdate)
          tc.podUpdateQueue.Done(podUpdate)
      }
  }
}
```
### 2、select中如何判断channel是关闭了并且不再读取

**如果case中的通道被关闭，读取会立即返回，将导致死循环。可以将通道置为nil来让select 忽略掉这个case**，继续评估其它case。但是有可能误判(写入方是写入了一个零值而不是关闭channel，比如整数0)

```go
for {
		select {
		case x, ok := <-chan1:
			fmt.Printf("%v,通道读取到：x=%v,ok=%v\n", time.Now().Format(timestamp), x, ok)
			time.Sleep(500 * time.Millisecond)
			if !ok {
				chan1 = nil
			}
		default:
			fmt.Printf("%v,通道未读取到，进入到default\n", time.Now().Format(timestamp))
			time.Sleep(500 * time.Millisecond)
		}
	}
```



## 五、相关wiki

- 非常经典 https://www.cnblogs.com/gongxianjin/p/17159582.html
