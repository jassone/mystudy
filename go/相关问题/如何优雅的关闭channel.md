## 如何优雅的关闭


## 一、channel关闭原则
- 关闭时机未知，在不改变channel自身状态的条件下，无法知道他是否已经关闭
- 不能无脑关闭，如果一个channel已经关闭，重复关闭channel会导致panic
- 往一个关闭的channel写数据，也会导致panic

根据上面的三个不易，可以整理出三条channel的关闭原则：

- 不要在消费者端关闭channel
- 不要在有多个并行的生产者时关闭channel
- **应该只在唯一或者最后一个生产者协程中关闭channel**

简单解释一下这三个原则：

- 第一个原则，消费者不知道不知道channel何时该关闭，如果关闭了已经关闭的channel，会导致panic，而且分布式应用通常有多个消费者，每个消费者的行为一致，这么多消费者都尝试关闭channel必然会导致panic
- 第二个原则，如果有多个生产者往channel中写数据，如果一个生产者关闭了channel，其他生产者还在往channel里写，必然会导致panic
- 第三个原则，只要坚持这个原则，就可以确保向已经关闭的channel中发送数据的情况不会发生

## 二、不优雅关闭channel

- defer+recover
- sys.Once，确保只会关闭一次
- sys.Mutex，确保只会关闭一次

## 三、优雅的关闭channel
根据生产者和消费者的数量关系，总共可以分为四种情况：

- 单生产者，单消费者
- 单生产者，多消费者
- 多生产者，单消费者
- 多生产者，多消费者

下面我们逐个分析这四种情况，应该如果优雅关闭channle

### 情况1：单生产者，单消费者
直接让生产者关闭channel即可

### 情况2：单生产者，多消费者
同样直接让生产者关闭channel即可

### 情况三：多生产者，单消费者/多消费者

##### 1.生产者会主动退出情况
```go
// 两个写，写10条数据，3个读，保证能读完

func main() {
   log.SetFlags(0)

   const NumReceivers = 3
   const NumSenders = 2
   const NumTotal = 10

   wg := sync.WaitGroup{}
   wg.Add(NumReceivers)

   dataCh := make(chan int)   // 将被关闭
   middleCh := make(chan int) // 不会被关闭
   closed := make(chan int)   //会被关闭

   // 中间层
   go func() {
      num := 0
      for {
         num++
         select {
         case v := <-middleCh:
            if num == NumTotal {
               close(closed) // 不再写
               dataCh <- v   // 将最后数据发生到通道
               close(dataCh) // 关闭数据通道
               return
            } else {
               dataCh <- v
            }
         }
      }
   }()

   // 发送者
   for i := 0; i < NumSenders; i++ {
      go func(i int) {
         for j := i * 5; j < i*5+50; j++ {
            select {
            case <-closed:
               return
            default:
            }

            select {
            case <-closed:
               return
            case middleCh <- j:
            }
         }
      }(i)
   }

   // 接收者
   for i := 0; i < NumReceivers; i++ {
      go func() {
         defer wg.Done()

         for value := range dataCh {
            log.Println(value)
         }
      }()
   }

   wg.Wait()

}
```

## 四、总结

- 要转换为关闭一个channel时，只能有一个写
- 增加一个channel控制另外一个channel关闭
- 让生产者和消费者的协程退出，因为没有对channel进行引用，channel会被GC回收

