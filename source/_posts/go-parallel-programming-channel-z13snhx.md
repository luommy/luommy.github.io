---
title: Go并发编程 | Channel
date: '2023-12-26 13:42:26'
updated: '2023-12-26 13:42:26'
excerpt: 从并发编程看channel
tags:
  - golang
  - 并发编程
categories:
  - Golang
  - 并发编程
permalink: /post/go-parallel-programming-channel-z13snhx.html
comments: true
toc: true
---



> 本部分的内容比较特殊，channel这部分我认为是很重要的，可以说是比较特殊的一种设计，除了晁老师的讲解我还会结合其他博主、专家的分析以查缺补漏。参考不限于小徐、小白、煎鱼等知名博主，希望站在巨人的肩上总结和深造。其实这些博主总结都很详细，希望日后有时间做一个专题纵向对比下，哪哪个关键点博主的文章更加透彻，在依据他们的总结做出我个人的理解和重点提取。
>
> 重要记录：
>
> * [ ] 2023.12 首次编写主要参考晁老师和《Go语言进阶之旅》，覆盖全面，建立了channel底层认知体系

> Channel
>
> Channel 是 Go 语言内建的 first-class 类型，也是 Go 语言与众不同的特性之一。
>
> Channel的设计来源于CSP理论，中文直译为通信顺序进程，或者叫做交换信息的循序进程，是用来描述并发系统中进行交互的一种模式。CSP 允许使用进程组件来描述系统，它们独立运行，并且只通过消息传递的方式通信。
>
> Channel 类型是 Go 语言内置的类型，你无需引入某个包，就能使用它。虽然 Go 也提供了传统的并发原语，但是它们都是通过库的方式提供的，你必须要引入 sync 包或者atomic 包才能使用它们，而 Channel 就不一样了，它是内置类型，使用起来非常方便。
>
> Channel 和 Go 的另一个独特的特性 goroutine 一起为并发编程提供了优雅的、便利的、与传统并发控制不同的方案，并演化出很多并发模式。

# CSP

‍

# Channel设计思想

<span style="font-weight: bold;" data-type="strong">Channel的设计思想：以通信方式共享内存,不要以共享内存方式通信</span>

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312261343631.png)​

# Channel使用场景

1. 数据交流：当作并发的 buffer 或者 queue，解决生产者 - 消费者问题。多个goroutine 可以并发当作生产者（Producer）和消费者（Consumer）。

2. 数据传递：一个 goroutine 将数据交给另一个 goroutine，相当于把数据的拥有权 (引用) 托付出去。

3. 信号通知：一个 goroutine 可以将信号 (closing、closed、data ready 等) 传递给另一个或者另一组 goroutine 。

4. 任务编排：可以让一组 goroutine 按照一定的顺序并发或者串行的执行，这就是编排的功能。

5. 锁：利用 Channel 也可以实现互斥锁的机制。

# Channel基本用法

有三种：只能发、只能收、既能发又能收。通常也可以说是两种，单向channel和双向channel。

​`ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType`​

习惯上将channel简写为chan，命名上也是沿用如此

正确示例：

```go
chan string  		// 可以发送接收string
chan<- struct{} 	// 只能发送struct{}
<-chan int 			// 只能从chan接收int
```

<u>*核心记忆法：*</u>​<u><span style="font-weight: bold;" data-type="strong">这个箭头总是射向左边的，元素类型总在最右边。如果箭头指向 chan，就表示可以往chan 中塞数据；如果箭头远离 chan，就表示 chan 会往外吐数据。</span></u> 

channel中的元素可以是任意类型：

```go
chan<- chan int
chan<- <-chan int
<-chan <-chan int
chan (<-chan int)
```

“<-”有个规则，总是尽量和左边的chan结合

```go
chan<- （chan int） 	   	// <- 和第一个chan结合
chan<- （<-chan int）  	// 第一个<-和最左边的chan结合，第二个<-和左边第二个chan结合
<-chan （<-chan int）  	// 第一个<-和最左边的chan结合，第二个<-和左边第二个chan结合
chan (<-chan int)     	// 因为括号的原因，<-和括号内第一个chan结合
```

有缓冲和无缓冲channel：

通过 make，我们可以初始化一个 chan，未初始化的 chan 的零值是 nil。你可以设置它的容量，比如下面的 chan 的容量是 9527，我们把这样的 chan 叫做 buffered chan；如果没有设置，它的容量是 0，我们把这样的 chan 叫做 unbuffered chan。

​`make(chan int, 9527)`​

如果 chan 中还有数据，那么，从这个 chan 接收数据的时候就不会阻塞，如果 chan 还未满（“满”指达到其容量），给它发送数据也不会阻塞，否则就会阻塞。unbuffered chan 只有读写都准备好之后才不会阻塞，这也是很多使用 unbuffered chan 时的常见Bug。

还有一个知识点需要记住：nil 是 chan 的零值，是一种特殊的 chan，对值是 nil 的 chan 的发送接收调用者总是会阻塞。

## 发送数据

```go
ch <- 2000
```

往 chan 中发送一个数据使用“ch<-”，发送数据是一条语句: 这里的 ch 是 chan int 类型或者是 chan <-int。

## 接收数据

从 chan 中接收一条数据使用“<-ch”，接收数据也是一条语句：

```go
x := <-ch 	// 把接收的一条数据赋值给变量x  2000
foo(<-ch) 	// 把接收的一个的数据作为参数传给函数 2000传给foo函数作为参数
<-ch 		// 丢弃接收的一条数据 2000丢了
```

这里的 ch 类型是 chan T 或者 <-chan T。接收数据时，还可以返回两个值。第一个值是返回的 chan 中的元素，很多人不太熟悉的是第二个值。第二个值是 bool 类型，代表是否成功地从 chan 中读取到一个值，如果第二个参数是 false，chan 已经被 close 而且 chan 中没有缓存的数据，这个时候，第一个值是零值。所以，如果从 chan 读取到一个零值，可能是 sender 真正发送的零值，也可能是closed 的并且没有缓存元素产生的零值。

## 如何解决零值问题？

在 Go 中，当从通道中接收到一个零值时，可能有多种原因。为了区分 sender 真正发送的零值和通道关闭时产生的零值，有几种方法可以尝试解决这个问题：

<span style="font-weight: bold;" data-type="strong">1. 使用额外的信号</span>

可以在发送零值之前在通道中发送一个额外的信号，比如使用一个布尔型的通道来表示是否关闭：

```go
ch := make(chan int)
closeFlag := make(chan bool)

// Sender
go func() {
    // Send zero value or actual value
    if value != 0 {
        ch <- value
    } else {
        close(ch)
        closeFlag <- true
    }
}()

// Receiver
select {
case value := <-ch:
    // Received value
    fmt.Println("Received:", value)
case <-closeFlag:
    // Channel closed without value
    fmt.Println("Channel closed")
}
```

<span style="font-weight: bold;" data-type="strong">2. 使用带缓冲的通道</span>

带缓冲的通道可以在关闭前允许缓存元素。通过检查通道的长度，可以区分发送的零值和通道关闭时产生的零值：

```go
ch := make(chan int, 1)

// Sender
go func() {
    if value != 0 {
        ch <- value
    } else {
        close(ch)
    }
}()

// Receiver
select {
case value := <-ch:
    // Received value
    fmt.Println("Received:", value)
default:
    // Channel is closed or no value received
    if len(ch) == 0 {
        fmt.Println("Channel closed without value")
    } else {
        // Process received value
        fmt.Println("Received:", <-ch)
    }
}
```

这些方法可以帮助区分发送的零值和通道关闭时产生的零值。

## 其他操作

Go 内建的函数 close、cap、len 都可以操作 chan 类型：close 会把 chan 关闭掉，cap返回 chan 的容量，len 返回 chan 中缓存的还未被取走的元素数量。

send 和 recv 都可以作为 select 语句的 case clause，如下面的例子：实际中并不常见

```go
func main() {
	var ch = make(chan int, 10)
	for i := 0; i < 10; i++ {
		select {
		case ch <- i:
		case v := <-ch:
			fmt.Println(v)
		}
	}
}
```

chan 还可以应用于 for-range 语句中，比如：

```go
for v := range ch {
	fmt.Println(v)
}
```

或者是忽略读取的值，只是清空 chan：

```go
for range ch {
}
```

在 Go 中，通道没有直接提供清空的内置方法。关于清空操作我的思考：

1. <span style="font-weight: bold;" data-type="strong">使用循环读取并丢弃通道中的值</span>：可以使用循环读取通道中的值，并不处理这些值，即丢弃它们。当通道中的所有值都被读取后，通道就变为空了。

    ```go
    for {
        select {
        case <-ch: // 丢弃通道中的值
            // Do nothing
        default:
            // 通道为空或已经被清空
            return
        }
    }
    ```
2. <span style="font-weight: bold;" data-type="strong">重新创建通道</span>：关闭现有的通道，然后创建一个新的通道。这种方式适用于不再需要之前通道中的值的情况。

    ```go
    func emptyChannel(ch chan int) chan int {
        close(ch)
        return make(chan int)
    }

    // 使用新的空通道
    newCh := emptyChannel(ch)
    ```

这些方法可以在不同情况下清空通道。但需要注意的是，直接关闭并重新创建通道会中断任何等待写入或读取的操作。因此，在应用这些方法时，需要小心谨慎，确保不会影响到正在进行的操作。

# Channel实现原理

要懂channel的实现原理必须懂其数据结构和基本大局，即channel的本质是一个有锁的环形队列。

## Channel数据结构

chan 类型的数据结构如下图所示，它的数据类型是runtime.hchan。 <span style="font-weight: bold;" data-type="strong">”</span> <span style="font-weight: bold;" data-type="strong">十一个字段+锁</span> <span style="font-weight: bold;" data-type="strong">“</span> 

```go
// src/runtime/chan.go
type hchan struct {
	qcount   uint    
	dataqsiz uint   
	buf      unsafe.Pointer 
	elemsize uint16
	closed   uint32
	elemtype *_type 
	sendx    uint  
	recvx    uint  
	recvq    waitq  
	sendq    waitq  

	lock mutex
}
```

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312261343771.png)​	

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312261343777.png)​

* qcount：代表 chan 中已经接收但还没被取走的元素的个数。内建函数 len 可以返回这个字段的值。同样可理解为队列中的元素总数量
* dataqsiz：循环队列的长度（大小）；**chan 使用一个循环队列来存放元素，循环队列很适合这种生产
  者 - 消费者的场景**（我很好奇为什么这个字段省略 size 中的 e）
* buf：【存放元素的循环队列的 buffer】指向长度为 dataqsiz 的底层数组，仅有当 channel 为缓冲型的才有意义
* elemtype 和 elemsize：chan 中元素的类型和 size。因为 chan 一旦声明，它的元素类型是固定的，即普通类型或者指针类型，所以元素大小也是固定的
* closed：是否关闭
* sendx：已发送元素在循环队列中的索引位置。处理发送数据的指针在 buf 中的位置。一旦接收了新的数据，指针就会加上elemsize，移向下一个位置。buf 的总大小是 elemsize 的整数倍，而且 buf 是一个循环列表
* recvx：已接收元素在循环队列中的索引位置。处理接收请求时的指针在 buf 中的位置。一旦取出数据，此指针会移动到下一个位置
* recvq：接受者的 sudog 等待队列（缓冲区不足时阻塞等待的 goroutine）。chan 是多生产者多消费者的模式，如果消费者因为没有数据可读而被阻塞了，就会被加入到 recvq 队列中
* sendq：发送者的 sudog 等待队列。如果生产者因为 buf 满了而阻塞，会被加入到 sendq 队列中

因此可见 recvq 和 sendq 是整个的核心，只要是channel阻塞了底层就是它们俩搞的

## Channel中真正的核心

经常会忽略一个点：`recvq`​和`sendq`​ 的数据类型——waitq，waitg的背后是sudog的结构

​`recvq`​ 和 `sendq`​，其表现为等待队列，其类型为 `runtime.waitq`​ 的双向链表结构：

```
type waitq struct {
	first *sudog
	last  *sudog
}
```

且无论是 `first`​ 属性又或是 `last`​，其类型都为 `runtime.sudog`​ 结构体：

```
type sudog struct {
	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer
	...
}
```

* g：指向当前的 goroutine。
* next：指向下一个 g。
* prev：指向上一个 g。
* elem：数据元素，可能会指向堆栈。

sudog 是 Go 语言中用于存放协程状态为阻塞的 goroutine 的双向链表抽象，你可以直接理解为一个正在等待的 goroutine 就可以了。

## Channel生命周期

要理解Channel 设计的完整核心无非就是按照它的使用流程（生命周期）——创建、发送、接受、关闭来推敲。头脑中清楚channel在这四个过程中大致做了什么工作，基本就能理解它了

### 创建Chan

核心就是：根据 chan 的容量的大小和元素的类型不同，初始化不同的存储空间

创建 channel 的演示代码：

```go
ch := make(chan string)
```

其在编译器翻译后对应 `runtime.makechan`​ 或 `runtime.makechan64`​ 方法：其实只关注 makechan 就好了，因为 makechan64 只是做了 size 检查，底层还是调用makechan 实现的

```go
// 通用创建方法
func makechan(t *chantype, size int) *hchan

// 类型为 int64 的进行特殊处理 缓冲区很大？！
func makechan64(t *chantype, size int64) *hchan
```

channel 的基本单位是 `hchan`​ 结构体，那么在创建 channel 时，究竟还需要做什么，这可以进一步帮助你理解其中的结构

分析一下 `makechan`​ 方法，就能知道了。

源码如下：

```go
// src/runtime/chan.go
func makechan(t *chantype, size int) *hchan {
	elem := t.elem
	// 检查代码
	mem, _ := math.MulUintptr(elem.size, uintptr(size))

	var c *hchan
	// 分配内存
	switch {
	case mem == 0:
		// chan的size或者元素的size是0，不必创建buf
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		c.buf = c.raceaddr()
	case elem.ptrdata == 0:
		// 元素不是指针，分配一块连续的内存给hchan数据结构和buf
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		// 元素包含指针，那么单独分配buf
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}
	// 元素大小、类型、容量都记录下来
	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)
	lockInit(&c.lock, lockRankHchan)

	return c
}
```

创建 channel 的逻辑主要分为三大块：

* 当前 channel 不存在缓冲区，也就是元素大小为 0 的情况下，就会调用 `mallocgc`​​ 方法分配一段连续的内存空间。
* 当前 channel 存储的类型存在指针引用，就会连同 `hchan`​​ 和底层数组同时分配一段连续的内存空间。
* 通用情况，即不包含指针的情况，默认分配相匹配的连续内存空间。

最终，针对不同的容量和元素类型，这段代码分配了不同的对象来初始化 hchan 对象的字段，返回 hchan 对象。

需要注意到一块特殊点，那就是 channel 的创建都是调用的 `mallocgc` 方法，也就是 channel 都是创建在堆上的。因此 channel 是会被 GC 回收的，自然也不总是需要 `close` 方法来进行显示关闭了。

从整体上来讲，`makechan`​ 方法的逻辑比较简单，就是创建 `hchan`​ 并分配合适的 `buf`​ 大小的堆上内存空间。

### 发送数据

channel 发送数据的演示代码：

```go
go func() {
    ch <- "加练"
}()
```

其在编译器翻译后对应 `runtime.chansend1`​ 方法：

```go
func chansend1(c *hchan, elem unsafe.Pointer) {
	chansend(c, elem, true, getcallerpc())
}
```

其作为编译后的入口方法，实则指向真正的实现逻辑，也就是 `chansend`​ 方法。

整体：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	// 第一部分，nil channel
	if c <span style="font-weight: bold;" class="mark"> nil {
		if !block {
			return false
		}
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	// 第二部分，如果chan没有被close,并且chan满了，直接返回
	if !block && c.closed </span> 0 && full(c) {
		return false
	}

	// 第三部分，chan已经被close的情景,再往里面发送数据的话会panic
	lock(&c.lock) // 开始加锁
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}

	// 第四部分，从接收队列中出队一个等待的receiver
	// 如果等待队列中有等待的 receiver，那么这段代码就把它从队列中弹出，
	// 然后直接把数据交给它（通过 memmove(dst, src, t.size)），而不需要放入到 buf 中，速度可以更快一些。
	if sg := c.recvq.dequeue(); sg != nil {
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}

	// 第五部分，buf还没满
	if c.qcount < c.dataqsiz {
		qp := chanbuf(c, c.sendx)
		if raceenabled {
			raceacquire(qp)
			racerelease(qp)
		}
		typedmemmove(c.elemtype, qp, ep)
		c.sendx++
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		c.qcount++
		unlock(&c.lock)
		return true
	}

	// 第六部分，buf满的情况。如果buf满了，发送者的 goroutine 就会加入到发送者的等待队列中，直到被唤醒。
	// 这个时候，数据或者被取走了，或者 chan 被 close 了。
	// chansend1不会进入if块里，因为chansend1的block=true
	if !block {
		unlock(&c.lock)
		return false
	}

	// 省略一些调试相关
	...
}

func full(c *hchan) bool {
	if c.dataqsiz <span style="font-weight: bold;" class="mark"> 0 {
		return c.recvq.first </span> nil
	}

	return c.qcount <span style="font-weight: bold;" class="mark"> c.dataqsiz
}
```

#### 前置处理

在第一部分中，我们先看看 chan 发送的一些前置判断和处理：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	// nil channel
	if c </span> nil {
		if !block {
			return false
		}
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}


	// 省略一些调试相关
	...
}

func full(c *hchan) bool {
	if c.dataqsiz <span style="font-weight: bold;" class="mark"> 0 {
		return c.recvq.first </span> nil
	}

	return c.qcount == c.dataqsiz
}
```

一开始 `chansend`​ 方法在会先判断当前的 channel 是否为 nil。若为 nil，在逻辑上来讲就是向 nil channel 发送数据，就会调用 `gopark`​ 方法使得当前 Goroutine 休眠，进而出现死锁崩溃，表象就是出现 `panic`​ 事件来快速失败。

紧接着会对非阻塞的 channel 进行一个上限判断，看看是否快速失败。

失败的场景如下：

* 若非阻塞且未关闭，同时底层数据 dataqsiz 大小为 0（缓冲区无元素），则会返回失败。-无缓冲管道写入失败
* 若是 qcount 与 dataqsiz 大小相同（缓冲区已满）时，则会返回失败。 -有缓存管道但缓存满了写入失败

#### 上互斥锁

在完成了 channel 的前置判断后，即将在进入发送数据的处理前，channel 会进行上锁：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	...
	lock(&c.lock)
}
```

上锁后就能保住并发安全。另外我们也可以考虑到，这种场景会相对依赖单元测试的覆盖，因为一旦没考虑周全，漏上锁了，基本就会出问题。

#### 直接发送

在正式开始发送前，加锁之后，会对 channel 进行一次状态判断（是否关闭）：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	...
	// chan已经被关闭了，这个时候写往里写入数据会panic，我相信大家会瞬间浮现报错的场景
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}

	// 存在阻塞过程中的缓冲数据
	if sg := c.recvq.dequeue(); sg != nil {
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}
}
```

这种情况是最为基础的，也就是当前 channel 有正在阻塞等待的接收方，那么只需要直接发送就可以了

#### 缓冲发送

非直接发送，那么就考虑第二种场景，判断 channel 缓冲区中是否还有空间：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	...
	if c.qcount < c.dataqsiz {
		qp := chanbuf(c, c.sendx)
		typedmemmove(c.elemtype, qp, ep)
		c.sendx++
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		c.qcount++
		unlock(&c.lock)
		return true
	}

	if !block {
		unlock(&c.lock)
		return false
	}
}
```

会对缓冲区进行判定（`qcount`​ 和 `dataqsiz`​ 字段），以此识别缓冲区的剩余空间。紧接进行如下操作：

* 调用 `chanbuf`​ 方法，以此获得底层缓冲数据中位于 sendx 索引的元素指针值。
* 调用 `typedmemmove`​ 方法，将所需发送的数据拷贝到缓冲区中。
* 数据拷贝后，对 sendx 索引自行自增 1。同时若 sendx 与 dataqsiz 大小一致，则归 0（环形队列）。
* 自增完成后，队列总数同时自增 1。解锁互斥锁，返回结果。

至此针对缓冲区的数据操作完成。但若没有走进缓冲区处理的逻辑，则会判断当前是否阻塞 channel，若为非阻塞，将会解锁并直接返回失败。

配合图示如下：

​​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312261343729.png)​​

#### 阻塞发送

在进行了各式各样的层层筛选后，接下来进入阻塞等待发送的过程：

```
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	...
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}

	mysg.elem = ep
	mysg.waitlink = nil
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.waiting = mysg
	gp.param = nil
	c.sendq.enqueue(mysg)

	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)

	KeepAlive(ep)
}
```

* 调用 `getg`​ 方法获取当前 goroutine 的指针，用于后续发送数据。
* 调用 `acquireSudog`​ 方法获取 `sudog`​ 结构体，并设置当前 sudog 具体的待发送数据信息和状态。
* 调用 `c.sendq.enqueue`​ 方法将刚刚所获取的 `sudog`​ 加入待发送的等待队列。
* 调用 `gopark`​ 方法挂起当前 goroutine（会记录执行位置），状态为 waitReasonChanSend，阻塞等待 channel。
* 调用 `KeepAlive`​ 方法保证待发送的数据值是活跃状态，也就是分配在堆上，避免被 GC 回收。

配合图示如下：

​![image](https://image.eddycjy.com/eff4c079cfd73b75e5b5f26fbac7fe1b.jpg)​

在当前 goroutine 被挂起后，其将会在 channel 能够发送数据后被唤醒：

```
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	...
	// 从这里开始唤醒，并恢复阻塞的发送操作
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if gp.param <span style="font-weight: bold;" class="mark"> nil {
		if c.closed </span> 0 {
			throw("chansend: spurious wakeup")
		}
		panic(plainError("send on closed channel"))
	}
	gp.param = nil
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	mysg.c = nil
	releaseSudog(mysg)
	return true
}
```

唤醒 goroutine（调度器在停止 g 时会记录运行线程和方法内执行的位置）并完成 channel 的阻塞数据发送动作后。进行基本的参数检查，确保是符合要求的（纵深防御），接着开始取消 mysg 上的 channel 绑定和 sudog 的释放。

至此完成所有类别的 channel 数据发送管理。
