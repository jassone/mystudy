## map底层原理

map 是一种key-value的键值对存储结构，其中key不能重复，底层用hash表存储。

## 一、结构 

map的数据结构在源码结构中的关键字段如下，在`src/runtime/map.go`中

```go
 type hmap struct {
    count     int    // 元素的个数
    // 代表哈希表中的元素个数，调用len(map)时，返回的就是该字段值。
    flags     uint8 
    // 状态标志（是否处于正在写入的状态等）
    B         uint8  
    // buckets（桶）的对数
    // 如果B=5，则buckets数组的长度 = 2^B=32，意味着有32个桶
    noverflow uint16 
    // 溢出桶的数量
    hash0     uint32 
    // 生成hash的随机数种子
    buckets    unsafe.Pointer 
    // 指向buckets数组的指针，数组大小为2^B，如果元素个数为0，它为nil。
    oldbuckets unsafe.Pointer 
    // 如果发生扩容，oldbuckets是指向老的buckets数组的指针，老的buckets数组大小是新的buckets的1/2;非扩容状态下，它为nil。
    nevacuate  uintptr        
    // 表示扩容进度，小于此地址的buckets代表已搬迁完成。
    extra *mapextra 
    // 存储溢出桶，这个字段是为了优化GC扫描而设计的，下面详细介绍
 }

type mapextra struct {
    //记录所有使用的溢出桶
    overflow    *[]*bmap 
    //用于在扩容阶段存储旧桶用到的溢出桶的地址
    oldoverflow *[]*bmap
    // 指向下一个空闲溢出桶
    nextOverflow *bmap
}
 
//每个bucket是一个bmap结构体，bmap 结构体如下
type bmap struct{
    tophash [8]uint8
    keys [8]keytype 
    // keytype 由编译器编译时候确定
    values [8]elemtype 
    // elemtype 由编译器编译时候确定
    overflow uintptr 
    // overflow指向下一个bmap，overflow是uintptr而不是*bmap类型，保证bmap完全不含指针，是为了减少gc，溢出桶存储到extra字段中
}
 
 //在编译期间会产生新的结构体
 type bmap struct {
     tophash [8]uint8 //存储哈希值的高8位
     data    byte[1]  //key value数据:key/key/key/.../value/value/value...
     overflow *bmap   //溢出bucket的地址
 }
```

为了便于理解源码的结构，我们提炼关键字段并转换为图形模式：

![1440w.webp](https://pic1.imgdb.cn/item/640a991ef144a010071bca25.webp)

![67f_1440w.webp](https://pic1.imgdb.cn/item/640aa336f144a010072ab455.webp)

在go的map实现中，它的底层结构体是hmap，hmap里维护着若干个bucket数组 (即桶数组)。

### 1、bmap结构

Bucket数组中每个元素都是bmap结构，也即每个bucket（桶）都是bmap结构，【ps：后文为了语义一致，和方便理解，就不再提bmap了，统一叫作桶】。

![8deed680fea38d_1440w.webp](https://pic1.imgdb.cn/item/640aa466f144a010072d57eb.webp)

bmap 结构体字段说明：

- tophash 用于快速查找key是否在该bucket中，在实现过程中会使用key的hash值的高8位作tophash值，存放在bmap tophash字段中。tophash字段不仅存储key哈希值的高8位，还会存储一些状态值，用来表明当前桶单元状态。
- bmap中的key和value是各自放在一起的，如k1/k2/...v1/v2...①，而不是key/value/key/value/...（kv形式）这样的形式，这样做的目的是节省空间。例如map[int64]int8，key是8字节，value是1字节，kv长度不同，如果按照②的kv形式存放，则需要考虑value也对齐占有8个字节，这样就会浪费7个字节的存储空间，所以go采用了第1种方式。
- 每个bucket最多只能放8个键值对，如果有第9个，落入当前的bucket，则需要再创建一个bucket，通过overflow指针连接起来。

## 二、map中数据操作

了解了map的数据结构后，下面让我们学习一下在map中存取数据的过程：

### 1、GET获取数据

**假设当前 B=4 即桶数量为2^B=16个**，要从map中获取k4对应的value

![eef50fdf76_1440w.webp](https://pic1.imgdb.cn/item/640a99faf144a010071cefc8.webp)

**参考上图，k4的get流程可以归纳为如下几步：**

1. **计算k4的hash值**。[由于当前主流机都是64位操作系统，所以计算结果有64个比特位]**
2. **通过最后的“B”位来确定在哪号桶**，此时B为4，所以取k4对应哈希值的后4位，也就是0101，0101用十进制表示为5，所以在5号桶）
3. **根据k4对应的hash值前8位快速确定是在这个桶的哪个位置**（额外说明一下，在bmap中存放了每个key对应的tophash，是key的哈希值前8位),一旦发现前8位一致，则会执行下一步
4. **对比key完整的hash是否匹配**，如果匹配则获取对应value
5. **如果都没有找到，就去连接的下一个溢出桶中找**

有很多同学会问这里为什么要多维护一个tophash，即hash前8位？

这是因为tophash可以快速确定key是否正确，也可以把它理解成一种缓存措施，如果前8位都不对了，后面就没有必要比较了。

##### 具体查找过程

![26ad18c5be27d5_1440w.webp](https://pic1.imgdb.cn/item/640ac40df144a01007684183.webp)

- 1、写保护监测

函数首先会检查 map 的标志位 flags。如果 flags 的写标志位此时被置 1 了，说明有其他协程在执行“写”操作，进而导致程序 panic，**这也说明了 map 不是线程安全的**。

```go
if h.flags&hashWriting != 0 {
    throw("concurrent map read and map write")
}
```

- 2、计算hash值

```go
hash := t.hasher(key, uintptr(h.hash0))               
//key经过哈希函数计算后，得到的哈希值如下（主流64位机下共 64 个 bit 位），不同类型的key会有不同的hash函数
10010111 | 000011110110110010001111001010100010010110010101010 │ 01010               
```

- 3、找到hash对应的bucket
  bucket定位：哈希值的低B个bit 位，用来定位key所存放的bucket。如果当前正在扩容中，并且定位到的旧bucket数据还未完成迁移，则使用旧的bucket（扩容前的bucket）

```go
hash := t.hasher(key, uintptr(h.hash0))
// 桶的个数m-1，即 1<<B-1,B=5时，则有0~31号桶
m := bucketMask(h.B)
// 计算哈希值对应的bucket
// t.bucketsize为一个bmap的大小，通过对哈希值和桶个数取模得到桶编号，通过对桶编号和buckets起始地址进行运算，获取哈希值对应的bucket
b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
// 是否在扩容
if c := h.oldbuckets; c != nil {
  // 桶个数已经发生增长一倍，则旧bucket的桶个数为当前桶个数的一半
    if !h.sameSizeGrow() {
        // There used to be half as many buckets; mask down one more power of two.
        m >>= 1
    }
    // 计算哈希值对应的旧bucket
    oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
    // 如果旧bucket的数据没有完成迁移，则使用旧bucket查找
    if !evacuated(oldb) {
        b = oldb
    }
}
```

- 4、遍历bucket查找
  tophash值定位：哈希值的高8个bit 位，用来快速判断key是否已在当前bucket中（如果不在的话，需要去bucket的overflow中查找）
  在 bucket 及bucket的overflow中寻找tophash 值（HOB hash）为 151* 的 槽位，即为key所在位置，找到了空槽位或者 2 号槽位，这样整个查找过程就结束了，其中找到空槽位代表没找到。

```go
for ; b != nil; b = b.overflow(t) {
        for i := uintptr(0); i < bucketCnt; i++ {
            if b.tophash[i] != top {
              // 未被使用的槽位，插入
                if b.tophash[i] == emptyRest {
                    break bucketloop
                }
                continue
            }
            // 找到tophash值对应的的key
            k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
            if t.key.equal(key, k) {
                e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
                return e
            }
        }
    }
```

bucket及tophash查找过程：

[![6c1c57d19_1440w.webp](https://pic1.imgdb.cn/item/6443a8ca0d2dde57771e4c23.webp)](https://pic1.imgdb.cn/item/6443a8ca0d2dde57771e4c23.webp)

- 5、返回key对应的指针
  如果通过上面的步骤找到了key对应的槽位下标 i，我们再详细分析下key/value值是如何获取的：

```go
// keys的偏移量
dataOffset = unsafe.Offsetof(struct{
  b bmap
  v int64
}{}.v)

// 一个bucket的元素个数
bucketCnt = 8

// key 定位公式
k :=add(unsafe.Pointer(b),dataOffset+i*uintptr(t.keysize))

// value 定位公式
v:= add(unsafe.Pointer(b),dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
```

bucket 里 keys 的起始地址就是 unsafe.Pointer(b)+dataOffset
第 i 个下标 key 的地址就要在此基础上跨过 i 个 key 的大小；
而我们又知道，value 的地址是在所有 key 之后，因此第 i 个下标 value 的地址还需要加上所有 key 的偏移。

### 2、PUT存放数据

**map的赋值流程可总结位如下几步：**

1. **通过key的hash值后“B”位确定是哪一个桶**，图中示例为4号桶。
2.  遍历当前桶，通过key的tophash和hash值，防止key重复，然后**找到第一个可以插入的位置**，即空位置处存储数据。
3. 如果**当前桶元素已满，会通过overflow链接创建一个新的桶**，来存储数据。

**关于hash冲突**：当两个不同的 key 落在同一个桶中，就是发生了哈希冲突。冲突的解决手段是采用链表法：在 桶 中，从前往后找到第一个空位进行插入。如果8个kv满了，那么当前桶就会连接到下一个溢出桶（bmap）。

## 三、扩容

### 1、扩容的方式

扩容有两种，一种是等量扩容，另一种是2倍扩容

**等量扩容**

由于map中不断的put和delete key，桶中可能会出现很多断断续续的空位，这些空位会导致连接的bmap溢出桶很长，导致扫描时间边长。这种扩容实际上是一种整理，把后置位的数据整理到前面。**这种情况下，元素会发生重排，但不会换桶。**

![061f67e41620b1504_1440w.webp](https://pic1.imgdb.cn/item/640a9be2f144a010071f7bbd.webp)

**2倍容量扩容**

这种2倍扩容是由于当前桶数组确实不够用了，**发生这种扩容时，元素会重排，可能会发生桶迁移**。

如图中所示，扩容前B=2,扩容后B=3，假设一元素key的hash值后三位为101，那么由上文的介绍可知，在扩容前，由hash值的后两位来决定几号桶，即 01 所以元素在1号桶。 在扩容发生后，由hash值得后三位来决定几号桶，即101所以元素会迁移到5号桶。

![904985a4ca1_1440w.webp](https://pic1.imgdb.cn/item/640aa21df144a0100729035d.webp)

### 2、发生扩容的条件

首先我们了解下**装载因子(loadFactor)**的概念

loadFactor:=count / (2^B) 即 装载因子 = map中元素的个数 / map中当前桶的个数

通过计算公式我们可以得知，**装载因子是指当前map中，每个桶中的平均元素个数。**

**扩容条件1**：**装载因子 > 6.5** (源码中定义的)

这个也非常容易理解，正常情况下，如果没有溢出桶，那么一个桶中最多有8个元素，当平均每个桶中的数据超过了6.5个，那就意味着当前容量要不足了，发生扩容。

**扩容条件2**: **溢出桶的数量过多**

当 B < 15 时，如果overflow的bucket数量超过 2^B。

当 B >= 15 时，overflow的bucket数量超过 2^15。

简单来讲，新加入key的hash值后B位都一样，使得个别桶一直在插入新数据，进而导致它的溢出桶链条越来越长。如此一来，当map在操作数据时，扫描速度就会变得很慢。及时的扩容，可以对这些元素进行重排，使元素在桶的位置更平均一些。

### 3、扩容时的细节

1. 在我们的hmap结构中有一个oldbuckets，扩容刚发生时，会先将老数据存到这个里面。
2. 每次对map进行删改操作时，会触发从oldbucket中迁移到bucket的操作【非一次性，分多次】
3. 在扩容没有完全迁移完成之前，每次get或者put遍历数据时，都会先遍历oldbuckets，然后再遍历buckets。

## 四、注意点

### 1、map线程并发问题

在同一时间点，两个 goroutine 对同一个map进行读写操作是不安全的(会panic)。这是由于golang的设计者考虑到使用map的场景都不是并发访问，如果map并发读写会产生什么呢？如果并发时写入，则会产生panic。runtime.map 代码判断：

```go
//赋值时检查是否在写入
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
    if h.flags&hashWriting != 0 {
        throw("concurrent map writes")
    }
}
//读取数据时检查是否在写入
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
     if h.flags&hashWriting != 0 {
        throw("concurrent map read and map write")
    }
 }
```

还有一种情况是**map是线程不安全的**，举个栗子：

某map桶数量为4，即B=2。此时 goroutine1来插入key1， goroutine2来读取 key2. 可能会发生如下过程：

1. goroutine2 计算key2的hash值,B=2，并确定桶号为1。
2. goroutine1添加key1，触发扩容条件。
3. B=B+1=3, buckets数据迁移到oldbuckets。
4. goroutine2从桶1中遍历，获取数据失败。

#####  如何安全使用map

Go map不是线程安全的，在使用过程中如果需要保证线程安全，则需要保持同步。

- 使用go官方提供的sync.Map替代map
- 使用sync.Mutex或sync.RWMutex进行加锁

```go
 type Resource struct {
     sync.RWMutex
     m map[string]int
 }
 
 func main() {
     r := Resource{m: make(map[string]int)}
 
     go func() { //开一个goroutine写map
         for j := 0; j < 100; j++ {
             r.Lock()
             r.m[fmt.Sprintf("resource_%d", j)] = j
             r.Unlock()
         }
     }()
     go func() { //开一个goroutine读map
         for j := 0; j < 100; j++ {
             r.RLock()
             fmt.Println(r.m[fmt.Sprintf("resource_%d", j)])
             r.RUnlock()
         }
     }()
 }
```

### 2、对map数据进行操作时不可取地址

因为随着map元素的增长，map底层重新分配空间会导致之前的地址无效。所以不允许取地址。也正因为如此，下面代码是错误的。

```go
type Student struct {
     name string
} 
func main() { 
    m := map[string]Student{"people": {"zhoujielun"}} 
    m["people"].name = "wuyanzu"
}
```

### 3、Go map遍历为什么是无序的

使用 range 多次遍历 map 时输出的 key 和 value 的顺序可能不同。这是 Go 语言的设计者们有意为之，旨在提示开发者们，Go 底层实现并不保证 map 遍历顺序稳定，不要依赖 range 遍历结果顺序。
主要原因有2点：

- map在遍历时，并不是从固定的0号bucket开始遍历的，每次遍历，都会从一个随机值序号的bucket，再从其中随机的cell开始遍历
- map遍历时，是按序遍历bucket，同时按需遍历bucket中和其overflow bucket中的cell。但是map在扩容后，会发生key的搬迁，这造成原来落在一个bucket中的key，搬迁后，有可能会落到其他bucket中了，从这个角度看，遍历map的结果就不可能是按照原来的顺序了。


因此如果不加入随机数，在不发生扩容情况下，一些不熟悉该原理的开发者会认为map是有序的，一旦依赖这个特性，就会引发bug。所以golang直接通过加随机数（在初始化迭代器时会生成一个随机数，决定从哪一个bucket开始迭代）避免问题的发生。这就是map为什么每次遍历顺序是不一样的原因。

**如何让map有序**
把key取出来进行排序，再通过key依次从map中取值。
