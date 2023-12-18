---
title: Go并发编程 | Mutex
date: '2023-12-06 18:14:22'
updated: '2023-12-07 14:07:41'
excerpt: 全面认识互斥锁Mutex
tags:
  - golang
  - 并发编程
categories:
  - Golang
  - 并发编程
permalink: /post/go-concurrent-programming-mutex-2f1e0r.html
comments: true
toc: true
---

# Go并发编程 | Mutex

> 本篇章志在全面、成体系的认识 Mutex，边学习的同时边总结，力求超越市面上99%的教程，超越的自信源于：全面、有简单应用、有深入源码、有演进历程、有个人理解和思考和持续维护持续修正、最重要的是站在巨人的肩膀上，汲取广大博主的分析精华。
>
> 认识到如何学好并发部分的源码：唯有加练，多看多画图，一遍不行就五遍，五遍不行就十遍。
>
> [画图工具](https://excalidraw.com/)

参考链接：

[案例](https://github.com/wuqinqiang/Go_Concurrency)

# 初识Mutex

> 多个 goroutine 并发更新同一个资源，像计数器；同时更新用户的账户信息；秒杀系统；往同一个 buffer 中并发写入数据等等。如果没有互斥控制，就会出现一些异常情况，比如计数器的计数不准确、用户的账户可能出现透支、秒杀系统出现超卖、buffer 中的数据混乱，等等，后果都很严重。

这些问题怎么解决呢？对，用互斥锁，那在 Go 语言里，就是 Mutex。

并发编程中涉及一个概念，叫做临界区。临界区就是一个被共享的资源，或者说是一个整体的一组共享资源，比如对数据库的访问、对某一个共享数据结构的操作、对一个 I/O 设备的使用、对一个连接池中的连接的调用，等等。为防止访问、操作错误，使用互斥锁，限定临界区只能同时由一个线程持有。

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926242.png)​

Mutex 是使用最广泛的同步原语（Synchronization primitives，有人也叫做并发原语）

同步原语适用场景：

<span style="font-weight: bold;" data-type="strong">共享资源</span>。并发地读写共享资源，会出现数据竞争（data race）的问题，所以需要Mutex、RWMutex 这样的并发原语来保护。

<span style="font-weight: bold;" data-type="strong">任务编排</span>。需要 goroutine 按照一定的规律执行，而 goroutine 之间有相互等待或者依赖的顺序关系，我们常常使用 WaitGroup 或者 Channel 来实现。

<span style="font-weight: bold;" data-type="strong">消息传递</span>。信息交流以及不同的 goroutine 之间的线程安全的数据交流，常常使用Channel 来实现。

同步原语没有固定标准的定义，通常下并发原语比同步原语更广一些，可以看做解决解决并发问题的一个数据结构。

简单来说，互斥锁 Mutex 就提供两个方法 Lock 和 Unlock：进入临界区之前调用 Lock

方法，退出临界区的时候调用 Unlock 方法：

​`func(m *Mutex)Lock()`​

​`func(m *Mutex)Unlock()`​

## 如果不用Mutex锁

创建了 10 个 goroutine，同时不断地对一个变量（count）进行加 1操作，每个 goroutine 负责执行 10 万次的加 1 操作，我们期望的最后计数的结果是 10 *100000 = 1000000 (一百万)。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var count = 0
	// 使用WaitGroup等待10个goroutine完成a
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			// 对变量count执行10次加1
			for j := 0; j < 100000; j++ {
				count++
			}
		}()
	}
	// 等待10个goroutine完成
	wg.Wait()
	fmt.Println(count)
}
	
```

使用 sync.WaitGroup 来等待所有的 goroutine 执行完毕后，再输出最终的结果。

可结果发现，每次结果都不是100w，且每次结果都不一样

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312071408947.png)​

count++ 不是一个原子操作，它至少包含几个步骤，比如读取变量count 的当前值，对这个值加 1，把结果再保存到 count 中。因为不是原子操作，就可能有并发的问题。

比如，10 个 goroutine 同时读取到 count 的值为 100，接着各自按照自己的逻辑加 1，值变成了 101，然后把这个结果再写回到 count 变量。但是，实际上，此时我们增加的总数应该是 10 才对，这里却只增加了 1，好多计数都被“吞”掉了。这是并发访问共享数据的常见错误。

```汇编
// count++操作的汇编代码
MOVQ "".count(SB), AX
LEAQ 1(AX), CX
MOVQ CX, "".count(SB)
```

## 并发问题排查工具race detector以及原理

在上述问题发生时，你可能很知道哪里出了问题，可是在实际业务环境下，可能很复杂，不容易被发现，

Go 提供了一个检测并发访问共享资源是否有问题的工具： race detector，它可以帮助我们自动发现程序有没有 data race 的问题。

Go race detector 是基于 Google 的 C/C++ sanitizers 技术实现的，编译器通过探测所有的内存访问，加入代码能监视对这些内存地址的访问（读还是写）。在代码运行的时候，race detector 就能监控到对共享变量的非同步访问，出现 race 的时候，就会打印出警告信息。

这个技术在 Google 内部帮了大忙，探测出了 Chromium 等代码的大量并发问题。Go 1.1中就引入了这种技术，并且一下子就发现了标准库中的 42 个并发问题。现在，race detector 已经成了 Go 持续集成过程中的一部分。

<span style="font-weight: bold;" data-type="strong"><u>*如何使用race detector？*</u></span>

在编译（compile）、测试（test）或者运行（run）Go 代码的时候，加上 race 参数，就

有可能发现并发问题。比如在上面的例子中，我们可以加上 race 参数运行，检测一下是不

是有并发问题。如果你 go run -race counter.go，就会输出警告信息。

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312071407613.png)​

这个警告不但会告诉你有并发问题，而且还会告诉你哪个 goroutine 在哪一行对哪个变量有写操作，同时，哪个 goroutine 在哪一行对哪个变量有读操作，就是这些并发的读写访问，引起了 data race。

虽然这个工具使用起来很方便，但是，因为它的实现方式，只能通过真正对实际地址进行读写访问的时候才能探测，所以它并不能在编译的时候发现 data race 的问题。而且，在运行的时候，只有在触发了 data race 之后，才能检测到，如果碰巧没有触发（比如一个data race 问题只能在 2 月 14 号零点或者 11 月 11 号零点才出现），是检测不出来的。

而且，把开启了 race 的程序部署在线上，还是比较影响性能的。运行 go tool compile -race -S counter.go，可以查看计数器例子的代码，重点关注一下 count++ 前后的编译后的代码

> ​`go tool compile -race -S counter.go`​ 是一个 Go 命令，用于编译 Go 源代码并生成汇编输出。它的各个部分含义如下：
>
> * ​`go tool compile`​: 这是 Go 编译器的命令行工具，用于将 Go 源代码编译成机器代码或汇编代码。
> * ​`-race`​: 这是一个编译器标志，表示启用 Go 的数据竞争检测器。数据竞争检测器可以帮助检测并发程序中的竞争条件。
> * ​`-S`​: 这是一个编译器标志，表示生成汇编代码输出。它让编译器生成对应输入源代码的汇编语言表示形式。

太少了，截取部分，在编译的代码中，增加了 runtime.racefuncenter、runtime.raceread、runtime.racewrite、runtime.racefuncexit 等检测 data race 的方法。通过这些插入的指令，Go race detector 工具就能够成功地检测出 data race 问题了。总结一下，通过在编译的时候插入一些指令，在运行时通过这些插入的指令检测并发读写从而发现 data race 问题，就是这个工具的实现机制。

```汇编
(base) PS E:\GoWork_K5\Try\Concurrency> go tool compile -race -S no-mutex.go
"".main STEXT size=478 args=0x0 locals=0x70 funcid=0x0 align=0x0
        0x0000 00000 (no-mutex.go:8)    TEXT    "".main(SB), ABIInternal, $112-0
        0x0000 00000 (no-mutex.go:8)    CMPQ    SP, 16(R14)
        0x0004 00004 (no-mutex.go:8)    PCDATA  $0, $-2
        0x0004 00004 (no-mutex.go:8)    JLS     468
        0x000a 00010 (no-mutex.go:8)    PCDATA  $0, $-1
        0x000a 00010 (no-mutex.go:8)    SUBQ    $112, SP
        0x000e 00014 (no-mutex.go:8)    MOVQ    BP, 104(SP)
        0x0013 00019 (no-mutex.go:8)    LEAQ    104(SP), BP
        0x0018 00024 (no-mutex.go:8)    FUNCDATA        $0, gclocals·0ce64bbc7cfa5ef04d41c861de81a3d7(SB)
        0x0018 00024 (no-mutex.go:8)    FUNCDATA        $1, gclocals·8757cff67ef48cad20738b76d6fada34(SB)
        0x0018 00024 (no-mutex.go:8)    FUNCDATA        $2, "".main.stkobj(SB)
        0x0018 00024 (no-mutex.go:8)    MOVQ    +112(FP), AX
        0x001d 00029 (no-mutex.go:8)    PCDATA  $1, $0
        0x001d 00029 (no-mutex.go:8)    NOP
        0x0020 00032 (no-mutex.go:8)    CALL    runtime.racefuncenter(SB)
        0x0025 00037 (no-mutex.go:9)    LEAQ    type.int(SB), AX
        0x002c 00044 (no-mutex.go:9)    CALL    runtime.newobject(SB)
        0x0031 00049 (no-mutex.go:9)    MOVQ    AX, "".&count+80(SP)
        0x0036 00054 (no-mutex.go:9)    PCDATA  $1, $1
        0x0036 00054 (no-mutex.go:9)    CALL    runtime.racewrite(SB)
        0x003b 00059 (no-mutex.go:9)    MOVQ    "".&count+80(SP), CX
        0x0040 00064 (no-mutex.go:9)    MOVQ    $0, (CX)
        0x0047 00071 (no-mutex.go:11)   LEAQ    type.sync.WaitGroup(SB), AX
        0x004e 00078 (no-mutex.go:11)   CALL    runtime.newobject(SB)
        0x0053 00083 (no-mutex.go:11)   MOVQ    AX, "".&wg+72(SP)
        0x0058 00088 (no-mutex.go:11)   MOVL    $16, BX
        0x005d 00093 (no-mutex.go:11)   PCDATA  $1, $2
        0x005d 00093 (no-mutex.go:11)   NOP
        0x0060 00096 (no-mutex.go:11)   CALL    runtime.racewriterange(SB)
        0x0065 00101 (no-mutex.go:11)   MOVQ    "".&wg+72(SP), AX
        0x006a 00106 (no-mutex.go:11)   MOVQ    $0, (AX)
        0x0071 00113 (no-mutex.go:11)   MOVL    $0, 8(AX)
        0x0078 00120 (no-mutex.go:12)   MOVL    $10, BX
        0x007d 00125 (no-mutex.go:12)   NOP
        0x0080 00128 (no-mutex.go:12)   CALL    sync.(*WaitGroup).Add(SB)
        0x0085 00133 (no-mutex.go:12)   XORL    AX, AX
        0x0087 00135 (no-mutex.go:13)   JMP     160
        0x0089 00137 (no-mutex.go:14)   CALL    runtime.newproc(SB)
        0x008e 00142 (no-mutex.go:13)   MOVQ    "".i+40(SP), CX
        0x0093 00147 (no-mutex.go:13)   LEAQ    1(CX), AX
```

## 使用mutex

这里的共享资源是 count 变量，临界区是 count++，只要在临界区前面获取锁，在离开临界区的时候释放锁，就能完美地解决 data race 的问题了。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	// 互斥锁保护计数器
	var mu sync.Mutex
	// 计数器的值
	var count = 0
	// 辅助变量，用来确认所有的goroutine都完成
	var wg sync.WaitGroup
	wg.Add(10)
	// 启动10个gourontine
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			// 累加10万次
			for j := 0; j < 100000; j++ {
				mu.Lock()
				count++
				mu.Unlock()
			}
		}()
	}
	wg.Wait()
	fmt.Println(count)
}

```

结果：

1000000

进程 已完成，退出代码为 0

这里有一点需要注意：Mutex 的零值是还没有 goroutine 等待的未加锁的状态，所以你不需要额外的初始化，直接声明变量（如 var mu sync.Mutex）即可。

## mutex常用操作

很多情况下，Mutex 会嵌入到其它 struct 中使用，比如下面的方式：

```go
type Counter struct {
	mu sync.Mutex //自动初始化为零值
	Count uint64
}
```

在初始化嵌入的 struct 时，也不必初始化这个 Mutex 字段，不会因为没有初始化出现空指针或者是无法获取到锁的情况。

关于Mutex结构体源码：

```go
// A Mutex is a mutual exclusion lock.
// The zero value for a Mutex is an unlocked mutex.
//
// A Mutex must not be copied after first use.
type Mutex struct {
	state int32
	sema  uint32
}

/* 在这个结构体中，有两个字段：

state int32：这个字段用于表示 Mutex 的状态。
它可能具有不同的状态，例如锁定状态或未锁定状态，
这里的具体数值含义会根据不同的操作系统和 CPU 架构有所不同。

sema uint32：这个字段是用于实现互斥锁的信号量。
sema 的值为 0 表示锁是未锁定状态。当值为 1 时表示锁是锁定状态。
这个字段与 state 一起协调实现了 Mutex 的锁定和解锁操作。

**/
```

将mutex嵌入struct中：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var counter Counter
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100000; j++ {
				counter.Lock()
				counter.Count++
				counter.Unlock()
			}
		}()
	}
	wg.Wait()
	fmt.Println(counter.Count)
}

type Counter struct {
	sync.Mutex
	Count uint64
}

```

如果嵌入的 struct 有多个字段，我们一般会把 Mutex 放在要控制的字段上面，然后使用空格把字段分隔开来。即使你不这样做，代码也可以正常编译，只不过，用这种风格去写的话，逻辑会更清晰，也更易于维护。

甚至，你还可以把获取锁、释放锁、计数加一的逻辑封装成一个方法，对外不需要暴露锁等逻辑：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	// 封装好的计数器
	var counter Counter
	var wg sync.WaitGroup
	wg.Add(10)
	// 启动10个goroutine
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			// 执行10万次累加
			for j := 0; j < 100000; j++ {
				counter.Incr() // +1操作受到锁保护的方法
			}
		}()
	}
	wg.Wait()
	fmt.Println(counter.Count())
}

// 线程安全的计数器类
type Counter struct {
	CounterType int
	Name        string
	mu          sync.Mutex
	count       uint64
}

// 加1的方法，内部使用互斥锁保护
func (c *Counter) Incr() {
	c.mu.Lock()
	c.count++
	c.mu.Unlock()
}

// 得到计数器的值，也需要锁保护，这么严谨的？！
func (c *Counter) Count() uint64 {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.count
}

```

Docker issue 37583、35517、32826、30696等、kubernetes issue72361、71617等，都是后来发现的 data race 而采用互斥锁 Mutex 进行修复的。这个后续有时间研究研究

‍

# Mutex 的实现及演进之路

理解Mutex的演进之路就能很清晰理解Mutex的设计思路！

如果一上来就读最新版的源码注定的孤独且枯燥的，很可能就放弃！

晁岳攀老师给出了“四个阶段”的Mutex演进架构：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926553.png)​

<span style="font-weight: bold;" data-type="strong">四个阶段：</span>

* “初版”的 Mutex 使用一个 flag 来表示锁是否被持有，实现比较简单；
* “给新人机会”：照顾到新来的 goroutine，所以会让新的 goroutine 也尽可能地先获取到锁；
* “多给些机会”：照顾新来的和被唤醒的 goroutine；
*    加入了饥饿的解决方案

## CAS

CAS（Compare and Swap，比较并交换）是一种并发算法，通常用于实现多线程环境下的同步操作。它是一种原子操作，用于在多线程环境下实现对内存中某个位置的值进行读取、比较和更新操作。

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926535.png)​

CAS 指令将给定的值和一个内存地址中的值进行比较，如果它们是同一个值，就使用新值替换内存地址中的值，这个操作是原子性的。那啥是原子性呢？如果你还不太理解这个概念，那么在这里只需要明确一点就行了，那就是原子性保证这个指令总是基于最新的值进行计算，如果<span style="font-weight: bold;" data-type="strong">同时有其它线程已经修改了这个值</span>，那么，CAS 会返回失败。

## 初版Mutex源码

如果你是设计者，怎么设计一个互斥锁？

> 可以通过一个 flag 变量，标记当前的锁是否被某个 goroutine 持有。
>
> 如果这个 flag 的值是 1，就代表锁已经被持有，那么，其它竞争的 goroutine 只能等待；
>
> 如果这个 flag 的值是 0，就可以通过 CAS（compare-and-swap，或者 compare-and-set）将这个 flag 设置为 1，标识锁被当前的这个 goroutine 持有了。

当然了对于我来说，一开始并不知道CAS，如果没有一定深度的并发基础我想也不会知道CAS。

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926255.png)​

‍

初版源码思路：

调用 Lock 请求锁的时候，通过 xadd 方法进行 CAS 操作（第 24 行），xadd 方法通过循环执行 CAS 操作直到成功，保证对 key 加 1 的操作成功完成。

如果比较幸运，锁没有被别的 goroutine 持有，那么，Lock 方法成功地将 key 设置为 1，这个 goroutine 就持有了这个锁；

如果锁已经被别的 goroutine 持有了，那么，当前的 goroutine 会把 key 加1，而且还会调用 semacquire 方法（第 27 行），使用信号量将自己休眠，等锁释放的时候，信号量会将它唤醒。

持有锁的 goroutine 调用 Unlock 释放锁时，它会将 key 减 1（第 31 行）。如果当前没有其它等待这个锁的 goroutine，这个方法就返回了。但是，如果还有等待此锁的其它goroutine，那么，它会调用 semrelease 方法（第 34 行），利用信号量唤醒等待锁的其它 goroutine 中的一个。

```go
// CAS操作，当时还没有抽象出atomic包
func cas(val *int32, old, new int32) bool {}
func semacquire(*int32) 
func semrelease(*int32) 

// 互斥锁的结构，包含两个字段
type Mutex struct {
	key  int32 // 锁是否被持有的标识
	sema int32 // 信号量专用，用以阻塞/唤醒goroutine
}

// 保证成功在val上增加delta的值
func xadd(val *int32, delta int32) (new int32) {
	for {
		v := *val
		if cas(val, v, v+delta) {
			return v + delta
		}
	}
	panic("unreached")
}

// 请求锁
func (m *Mutex) Lock() {
	if xadd(&m.key, 1) <span style="font-weight: bold;" class="mark"> 1 { //标识加1，如果等于1，成功获取到锁
		return
	}
	semacquire(&m.sema) // 否则阻塞等待
}
func (m *Mutex) Unlock() {
	if xadd(&m.key, -1) </span> 0 { // 将标识减去1，如果等于0，则没有其它等待者
		return
	}
	semrelease(&m.sema) // 唤醒其它阻塞的goroutine
}

```

所以，到这里，我们就知道了，初版的 Mutex 利用 CAS 原子操作，对 key 这个标志量进行设置。key 不仅仅标识了锁是否被 goroutine 所持有，还记录了当前持有和等待获取锁的 goroutine 的数量

※初版源码※ 其实就可以分析出一个问题：

* Unlock 方法可以被任意的 goroutine 调用释放锁，即使是没持有这个互斥锁的goroutine，也可以进行这个操作。

这是因为，<u>*Mutex 本身并没有包含持有这把锁的goroutine 的信息*</u>，所以，Unlock 也不会对此进行检查。Mutex 的这个设计一直保持至今。

所以，在实际应用中，强调锁要成对出现，而且不要离得太远，遵循“谁申请、谁释放”原则。

不要在一个方法中单独申请锁，而在另外一个方法中单独释放锁，一般都会在同一个方法中获取锁和释放锁。

例如：

```go
type Foo struct {
	mu sync.Mutex
	count int
}

func (f *Foo) Bar() {
	f.mu.Lock()

	if f.count < 1000 {
		f.count += 3
		f.mu.Unlock() // 此处释放锁
		return
	}

	f.count++
	f.mu.Unlock() // 此处释放锁
	return
}
```

从 1.14 版本起，Go 对 defer 做了优化，采用更有效的内联方式，取代之前的生成 defer对象到 defer chain 中，defer 对耗时的影响微乎其微了，所以基本上修改成下面简洁的写法：

```go
func (f *Foo) Bar() {
	f.mu.Lock()
	defer f.mu.Unlock()

	if f.count < 1000 {
		f.count += 3
		return
	}

	f.count++
	return
}
```

Lock/Unlock 总是成对紧凑出现，不会遗漏或者多调用，代码更少、更易理解。

初版的 Mutex 实现之后，Go 开发组又对 Mutex 做了一些微调，比如把字段类型变成了uint32 类型；调用 Unlock 方法会做检查；使用 atomic 包的同步原语执行原子操作等等，这些小的改动，都不是核心功能。

初版的时候可能核心人员就发现了一个核心问题：请求锁的 goroutine 会排队等待获取互斥锁。虽然这貌似很公平，但是从性能上来看，却不是最优的。因为如果我们能够把锁交给正在占用 CPU 时间片的 goroutine 的话，那就不需要做上下文的切换，在高并发的情况下，可能会有更好的性能。

有一说一如果是我，我是根本想不到这层次的。

## 二版：新goroutine也有机会

### Mutex结构体变化

```go
type Mutex struct {
	state int32  // 将key变为了state，这一个字段被分成了三部分，代表三个数据，这个是变得复杂的核心
	sema uint32  // sema主要内部用来阻塞等待的队列
}

// 这三个字段是真的究极难懂.
// 一个字段包含多个意义，这样可以通过尽可能少的内存来实现互斥锁。
// 第一位（最小的一位）mutexLocked 来表示这个锁是否被持有
// 第二位 mutexWoken 代表是否有唤醒的 goroutine
// 第三位 mutexWaiterShift 代表的是等待此锁的 goroutine 数量
const (
	mutexLocked = 1 << iota // mutex is locked   是否持有锁
	mutexWoken				// 是否被唤醒的标记
	mutexWaiterShift = iota // 阻塞等待的waiter的数量
)
```

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926053.png)​

### 请求锁Lock：流程更加复杂

<span style="font-weight: bold;" data-type="strong">源码：</span>

1. 首先，它尝试通过 `atomic.CompareAndSwapInt32`​ 直接获取锁，也就是将锁的状态从 0 修改为 `mutexLocked`​。
2. 如果获取锁失败（因为有其他 goroutine 持有了锁），它会进入一个循环。在循环中，它会尝试修改状态以请求锁。
3. 如果旧的状态中锁是已经被持有的（`old&mutexLocked != 0`​），那么会增加等待者的数量。
4. 如果 goroutine 是被唤醒的（标志为 `awoke`​），它会清除唤醒标志。
5. 最后，它会通过 `atomic.CompareAndSwapInt32`​ 设置新的状态。如果锁之前的状态是未加锁的，则退出循环并成功获取了锁；否则，它会请求信号量（`runtime.Semacquire(&m.sema)`​）来等待锁的释放。

```go
func (m *Mutex) Lock() {
	// Fast path: 幸运case，能够直接获取到锁
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		return
	}

	awoke := false
	for {
		old := m.state
		new := old | mutexLocked // 新状态加锁
		if old&mutexLocked != 0 {
			new = old + 1<<mutexWaiterShift //等待者数量加一
		}
		if awoke {
			// goroutine是被唤醒的，
			// 新状态清除唤醒标志
			new &^= mutexWoken
		}
		if atomic.CompareAndSwapInt32(&m.state, old, new) {//设置新状态
			if old&mutexLocked == 0 { // 锁原状态未加锁
				break
			}
			runtime.Semacquire(&m.sema) // 请求信号量
			awoke = true
		}
	}
}
```

如果想要获取锁的 goroutine 没有机会获取到锁，就会进行休眠，但是在锁释放唤醒之后，它并不能像先前一样直接获取到锁，还是要和正在请求锁的 goroutine 进行竞争。这会给后来请求锁的 goroutine 一个机会，也让 CPU 中正在执行的 goroutine 有更多的机会获取到锁，在一定程度上提高了程序的性能。

核心分类图：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926768.png)​

请求锁的 goroutine 有两类，一类是新来请求锁的 goroutine，另一类是被唤醒的等待请

求锁的 goroutine。锁的状态也有两种：加锁和未加锁。我用一张表格，来说明一下

goroutine 不同来源不同状态下的处理逻辑

### 释放锁Unlock：也很复杂

<span style="font-weight: bold;" data-type="strong">源码：</span>

```go
func (m *Mutex) Unlock() {
	// Fast path: drop lock bit.
	// 尝试将持有锁的标识设置为未加锁的状态，这是通过减 1 而不是将标志位置零的方式实现。
	new := atomic.AddInt32(&m.state, -mutexLocked) //去掉锁标志
	// 会检测原来锁的状态是否未加锁的状态，如果是 Unlock 一个未加锁的 Mutex 会直接 panic。
	if (new+mutexLocked)&mutexLocked == 0 { //本来就没有加锁
		panic("sync: unlock of unlocked mutex")
	}

	// 到这里为什么不直接返回？
	// 因为还可能有一些等待这个锁的 goroutine（有时候我也把它们称之为 waiter）需要通过信号量的方式唤醒它们中的一个。

	old := new
	for {
		// 如果等待的锁的goroutine为0、或者是否有锁、是否被唤醒之一，满足的话直接返回
		if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken) != 0
			return
		}
		new = (old - 1<<mutexWaiterShift) | mutexWoken // 新状态，准备唤醒gor
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			runtime.Semrelease(&m.sema)
			return
		}
		old = m.state
	}

/*
	这里晁老师说明有两种情况:
	第一种情况，如果没有其它的 waiter，说明对这个锁的竞争的 goroutine 只有一个，那就可以直接返回了；
如果这个时候有唤醒的 goroutine，或者是又被别人加了锁，那么，无需我们操劳，其它 goroutine 自己干得都很好，当前的这个 goroutine 就可以放心返回
了。
	第二种情况，如果有等待者，并且没有唤醒的 waiter，那就需要唤醒一个等待的 waiter。
在唤醒之前，需要将 waiter 数量减 1，并且将 mutexWoken 标志设置上，这样，Unlock就可以返回了。
*/

}
```

更好的理解上述源码你要知道的内容：

1. 如何理解CSA升级后的`atomic.AddInt32`​操作？

这段代码中的 `new := atomic.AddInt32(&m.state, -mutexLocked)`​ 是使用原子操作来修改 `m.state`​ 中存储的锁状态信息。这里使用了 `AddInt32`​ 函数将 `mutexLocked`​ 的负值加到 `m.state`​ 上，实现了去掉锁标志的操作。

在 `mutexLocked`​ 被定义为 `1 << iota`​ 的情况下，它的值是 `1`​。因此 `-mutexLocked`​ 的值就是 `-1`​，它在二进制补码中表示为所有位都为 `1`​。

假设 `m.state`​ 的初始值是 `mutexLocked`​，即锁已被获取，二进制表示为 `0001`​。

执行 `new := atomic.AddInt32(&m.state, -mutexLocked)`​ 时，会将 `-mutexLocked`​（即 `-1`​，二进制中所有位为 `1`​）加到 `m.state`​ 上：

```plaintext
   0001  (m.state)
+  1111  (-mutexLocked)
-----------
   0000  (new)
```

这样就通过原子操作将 `m.state`​ 的值从 `mutexLocked`​（表示锁已被获取）修改为 `0`​（表示未加锁状态）

2. ​`for { ... }`​：这是一个无限循环，表明会一直执行里面的逻辑直到满足某个条件才会退出

3. for循环中第一种情况关于`old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken) != 0`​的理解？

    * ​`old>>mutexWaiterShift == 0`​ 右移位运算，检查等待的goroutine数量是否为零。如果为 `0`​，意味着没有等待的 goroutine。
    * ​`old&(mutexLocked|mutexWoken) != 0`​ 检查 `old`​ 中是否包含锁定标志或唤醒标志，如果结果不为 `0`​，表示已经有其他线程持有锁或者有 goroutine 被唤醒了，因此不需要进一步唤醒等待的 goroutine。
    * 如果上述条件成立，即等待的 goroutine 数量为 `0`​ 或者已经有其他线程持有锁或者有 goroutine 被唤醒，那么结束循环，Unlock可以返回了。

4. for循环中关于第二种情况`new = (old - 1<<mutexWaiterShift) | mutexWoken`​的理解？

    * 执行到这里代表上述条件不成立，需要重新计算new值
    * 如果有等待者，并且没有唤醒的 waiter，那就需要唤醒一个等待的 waiter。在唤醒之前，需要将 waiter 数量减 1，并且将 mutexWoken 标志设置上，这样，Unlock就可以返回了。
    * ​`new = (old - 1<<mutexWaiterShift) | mutexWoken`​：计算新的状态，其中包括减少等待者数量并设置唤醒标志，以准备唤醒等待的 goroutine。
    * ​`atomic.CompareAndSwapInt32(&m.state, old, new)`​：使用原子操作比较并交换 `m.state`​ 的值，将 `old`​ 替换为 `new`​。这是为了确保在并发情况下，`m.state`​ 没有被其他地方修改过。如果成功替换，则释放信号量并返回，表示成功唤醒等待的 goroutine。如果替换失败，说明 `m.state`​ 已经被其他操作修改过，需要重新获取最新的 `m.state`​ 值进行下一轮的检查和操作。
5. 关于解锁部分的原子操作：

    使用 `atomic.CompareAndSwapInt32`​ 进行原子操作，尝试将 `m.state`​ 的值从 `old`​ 更新为 `new`​。如果这个操作成功，表示成功修改了状态，接着会释放信号量（`Semrelease`​）并返回，表示成功唤醒等待的 goroutine。如果更新失败，意味着 `m.state`​ 已经被其他线程修改过，所以重新获取最新的 `m.state`​ 值作为 `old`​，继续循环尝试更新状态。

总体来说，这段 for 循环的核心就是：通过不断地检查 `m.state`​ 的状态，如果满足某些条件（没有等待的 goroutine 或者已经有其他线程持有锁或有 goroutine 被唤醒），则直接返回；如果条件不满足，则尝试修改状态并唤醒等待的 goroutine。

相对于初版的设计，这次的改动主要就是，新来的 goroutine 也有机会先获取到锁，甚至<span style="font-weight: bold;" data-type="strong">一个 goroutine 可能连续获取到锁，打破了先来先得的逻辑</span>。但是，代码复杂度也显而易见。虽然这一版的 Mutex 已经给新来请求锁的 goroutine 一些机会，让它参与竞争，没有空的锁或者竞争失败才加入到等待队列中。但是其实还可以进一步优化。

如果说要彻底的弄明白，可能需要完整地画下图，模拟下计算过程。比如位运算部分为什么要那么计算。

## 三版：多给些机会

在 2015 年 2 月的改动中，如果新来的 goroutine 或者是被唤醒的 goroutine 首次获取不到锁，它们就会通过自旋（spin，通过循环不断尝试，spin 的逻辑是在runtime 实现的）的方式，尝试检查锁是否被释放。在尝试一定的自旋次数后，再执行原来的逻辑。

<span style="font-weight: bold;" data-type="strong">源码：</span>

```go
func (m *Mutex) Lock() {
	// Fast path: 幸运之路，正好获取到锁
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		return
	}
	awoke := false
	iter := 0
	for { // 不管是新来的请求锁的goroutine, 还是被唤醒的goroutine，都不断尝试请求锁
		old := m.state // 先保存当前锁的状态
		new := old | mutexLocked // 新状态设置加锁标志
		if old&mutexLocked != 0 { // 锁还没被释放
			// 新引入了自旋！自旋等待是为了减少因等待锁而发生的上下文切换带来的开销
			if runtime_canSpin(iter) { 
				if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0
					atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken)
					awoke = true
				}
				runtime_doSpin()
				iter++
				continue // 自旋，再次尝试请求锁
			}
			new = old + 1<<mutexWaiterShift
		}

		if awoke { // 唤醒状态
			// 新的锁的状态异常，这个可能存在吗？
			if new&mutexWoken == 0 {
				panic("sync: inconsistent mutex state")
			}
			new &^= mutexWoken // 新状态清除唤醒标记
		}
		// 注意iter=0
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			if old&mutexLocked == 0 { // 旧状态锁已释放，新状态成功持有了锁，直接
				break
			}
			runtime_Semacquire(&m.sema) // 阻塞等待
			awoke = true // 被唤醒
			iter = 0
		}
	}
}
```

这次的优化，增加了第 13 行到 21 行、第 25 行到第 27 行以及第 39 行。我来解释一下主要的逻辑，也就是第 13 行到 21 行。如果可以 spin 的话，第 9 行的 for 循环会重新检查锁是否释放。对于临界区代码执行非常短的场景来说，这是一个非常好的优化。因为临界区的代码耗时很短，锁很快就能释放，而抢夺锁的 goroutine 不用通过休眠唤醒方式等待调度，直接 spin 几次，可能就获得了锁。

关于自旋的理解：

​`iter`​ 是迭代计数器，它记录了自旋的次数或轮数

* ​`runtime_doSpin()`​：模拟实现自旋的函数。在这个函数中，会进行一些空循环，以达到自旋的目的。
* ​`iter++`​：迭代计数器加一，用于记录自旋的次数
* ​`continue`​：继续下一次循环，继续尝试请求锁，即重新进入自旋状态

进入自旋的条件：

* ​`!awoke`​：确保在被唤醒后不再尝试自旋等待
* ​`old&mutexWoken == 0`​：检查当前 Mutex 的状态是否已经有其他 goroutine 进行了唤醒操作。如果已经被唤醒，则不再进行自旋
*  `old>>mutexWaiterShift != 0`​，目的是检查是否有等待的 goroutine

## 终极版：解决饥饿问题

经过几次优化，Mutex 的代码越来越复杂，应对高并发争抢锁的场景也更加公平。但是总有极端情况，因为新来的 goroutine 也参与竞争，有可能每次都会被新来的 goroutine 抢获取锁的机会，在极端情况下，等待中的 goroutine 可能会一直获取不到锁，这就是饥饿问题。

2016 年 Go 1.9 中 Mutex 增加了饥饿模式，让锁变得更公平，不公平的等待时间限制在 1 毫秒，并且修复了一个大 Bug：总是把唤醒的goroutine 放在等待队列的尾部，会导致更加不公平的等待时间。

之后，2018 年，Go 开发者将 fast path 和 slow path 拆成独立的方法，以便内联，提高性能。2019 年也有一个 Mutex 的优化，虽然没有对 Mutex 做修改，但是，对于 Mutex唤醒后持有锁的那个 waiter，调度器可以有更高的优先级去执行，这已经是很细致的性能优化了。

没错，这一次优化添加了一种状态模式到state中：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926582.png)​

### 核心思路

Mutex 绝不容忍一个goroutine 被落下，永远没有机会获取锁。不抛弃不放弃是它的宗旨，而且它也<span style="font-weight: bold;" data-type="strong">尽可能地让等待较长的 goroutine 更有机会获取到锁。</span>

这个时候可以脑部一下，整体的逻辑不是又多了很多很多的分支？

即使不懂源码，也可以轻易的清楚一条链路：在除了很幸运地直接获取锁之外，还有自旋竞争之路、自旋竞争时间太长导致的饥饿之路。

饥饿时间阈值为10 的 6 次方纳秒，也就是1毫秒。

### 饥饿模式与正常模式的转换

饥饿--->正常：

正常模式下，waiter 都是进入先入先出队列，被唤醒的 waiter 并不会直接持有锁，而是要和新来的 goroutine 进行竞争。新来的 goroutine 有先天的优势，它们正在 CPU 中运行，可能它们的数量还不少，所以，在高并发情况下，被唤醒的 waiter 可能比较悲剧地获取不到锁，这时，它会被插入到队列的前面。如果 waiter 获取不到锁的时间超过阈值 1 毫秒，那么，这个 Mutex 就进入到了饥饿模式。

在饥饿模式下，Mutex 的拥有者将直接把锁交给队列最前面的 waiter。新来的 goroutine不会尝试获取锁，即使看起来锁没有被持有，它也不会去抢，也不会 spin，它会乖乖地加入到等待队列的尾部。

正常--->饥饿：

此 waiter 已经是队列中的最后一个 waiter 了，没有其它的等待锁的 goroutine 了；

此 waiter 的等待时间小于 1 毫秒。

---

正常模式拥有更好的性能，因为即使有等待抢锁的 waiter，goroutine 也可以连续多次获取到锁。饥饿模式是对公平性和性能的一种平衡，它避免了某些 goroutine 长时间的等待锁。在饥饿模式下，优先对待的是那些一直在等待的 waiter。

### 饥饿源码

正常模式下，waiter 都是进入先入先出队列，被唤醒的 waiter 并不会直接持有锁，而是要和新来的 goroutine 进行竞争。新来的 goroutine 有先天的优势，它们正在 CPU 中运行，可能它们的数量还不少，所以，在高并发情况下，被唤醒的 waiter 可能比较悲剧地获取不到锁，这时，它会被插入到队列的前面。

如果 waiter 获取不到锁的时间超过阈值 1 毫秒，那么，这个 Mutex 就进入到了饥饿模式。

在饥饿模式下，Mutex 的拥有者将直接把锁交给队列最前面的 waiter。新来的 goroutine不会尝试获取锁，即使看起来锁没有被持有，它也不会去抢，也不会 spin，它会乖乖地加入到等待队列的尾部。如果拥有 Mutex 的 waiter 发现下面两种情况的其中之一，它就会把这个 Mutex 转换成正常模式:正常模式拥有更好的性能，因为即使有等待抢锁的 waiter，goroutine 也可以连续多次获取到锁。

饥饿模式是对公平性和性能的一种平衡，它避免了某些 goroutine 长时间的等待锁。在饥饿模式下，优先对待的是那些一直在等待的 waiter。

```go
type Mutex struct {
	state int32
	sema uint32
}

const (
	mutexLocked = 1 << iota // mutex is locked
	mutexWoken
	mutexStarving // 从state字段中分出一个饥饿标记
	mutexWaiterShift = iota
	starvationThresholdNs = 1e6
)

func (m *Mutex) Lock() {
	// Fast path: 幸运之路，一下就获取到了锁
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
	return
	}
	// Slow path：缓慢之路，尝试自旋竞争或饥饿状态下饥饿goroutine竞争
	m.lockSlow()
}

func (m *Mutex) lockSlow() {
	var waitStartTime int64
	starving := false // 此goroutine的饥饿标记
	awoke := false // 唤醒标记
	iter := 0 // 自旋次数
	old := m.state // 当前的锁的状态
	for {
		// 锁是非饥饿状态，锁还没被释放，尝试自旋
		if old&(mutexLocked|mutexStarving) <span style="font-weight: bold;" class="mark"> mutexLocked && runtime_canSp
			if !awoke && old&mutexWoken </span> 0 && old>>mutexWaiterShift != 0
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken)
				awoke = true
			}
			runtime_doSpin()
			iter++
			old = m.state // 再次获取锁的状态，之后会检查是否锁被释放了
			continue
		}
		new := old
		if old&mutexStarving == 0 {
			new |= mutexLocked // 非饥饿状态，加锁
		}
		if old&(mutexLocked|mutexStarving) != 0 {
			new += 1 << mutexWaiterShift // waiter数量加1
		}
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving // 设置饥饿状态
		}
		if awoke {
			if new&mutexWoken == 0 {
				throw("sync: inconsistent mutex state")
			}
			new &^= mutexWoken // 新状态清除唤醒标记
		}
		// 成功设置新状态
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			// 原来锁的状态已释放，并且不是饥饿状态，正常请求到了锁，返回
			if old&(mutexLocked|mutexStarving) == 0 {
				break // locked the mutex with CAS
			}
			// 处理饥饿状态
			// 如果以前就在队列里面，加入到队列头
			queueLifo := waitStartTime != 0
			if waitStartTime == 0 {
				waitStartTime = runtime_nanotime()
			}
			// 阻塞等待
			runtime_SemacquireMutex(&m.sema, queueLifo, 1)
			// 唤醒之后检查锁是否应该处于饥饿状态
			starving = starving || runtime_nanotime()-waitStartTime > star
			old = m.state
			// 如果锁已经处于饥饿状态，直接抢到锁，返回
			if old&mutexStarving != 0 {
				if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterSh
					throw("sync: inconsistent mutex state")
				}
				// 有点绕，加锁并且将waiter数减1
				delta := int32(mutexLocked - 1<<mutexWaiterShift)
				if !starving || old>>mutexWaiterShift == 1 {
					delta -= mutexStarving // 最后一个waiter或者已经不饥饿了，清
				}
				atomic.AddInt32(&m.state, delta)
				break
			}
			awoke = true
			iter = 0
		} else {
			old = m.state
		}
	}
}

func (m *Mutex) Unlock() {
	// Fast path: drop lock bit.
	new := atomic.AddInt32(&m.state, -mutexLocked)
	if new != 0 {
		m.unlockSlow(new)
	}
}

func (m *Mutex) unlockSlow(new int32) {
	if (new+mutexLocked)&mutexLocked <span style="font-weight: bold;" class="mark"> 0 {
		throw("sync: unlock of unlocked mutex")
	}
	if new&mutexStarving </span> 0 {
		old := new
		for {
			if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|m
				return
			}
			new = (old - 1<<mutexWaiterShift) | mutexWoken
			if atomic.CompareAndSwapInt32(&m.state, old, new) {
				runtime_Semrelease(&m.sema, false, 1)
				return
			}
			old = m.state
		}
	} else {
		runtime_Semrelease(&m.sema, true, 1)
	}
```

Go 语言从出生到现在已经 10 多年了，这个 Mutex 对外的接口却没有变化，依然向下兼容，即使现在 Go 出了两个版本，每个版本也会向下兼容，保持 Go 语言的稳定性，你也能领悟他们软件开发和设计的思想。还有一点，你也可以观察到，为了一个程序 20% 的特性，你可能需要添加 80% 的代码，这也是程序越来越复杂的原因。所以，最开始的时候，如果能够有一个清晰而且易于扩展的设计，未来增加新特性时，也会更加方便。

最后，两个小问题：

1. 目前 Mutex 的 state 字段有几个意义，这几个意义分别是由哪些字段表示的？

	4个；

	mutexLocked代表是否被锁上 mutexWoken代表唤醒标记

	mutexStarving代表是否处于饥饿 mutexWaiterShift 代表有多少go程在等待这个锁

2. 等待一个 Mutex 的 goroutine 数最大是多少？是否能满足现实的需求？

	state 是int32，有三位用来表示状态的，剩下的用来表示数量，减去状态位所以能够表示的最大值

	是2^32-3^-1如果每个等待这个 Mutex 的 goroutine 占用 2KB 内存，那么最大约能支持 2KB *

	1073741822 ≈1048575M≈1024G≈1T 的内存占用。这个数量对于绝大多数应用场景来说已经非

	常巨大了，可以满足绝大多数实际需求。

# Mutex四大易错场景

使用mutex的过程中可能会犯一些错误

总体来说有四方面：

* Lock/Unlock 不是成对出现
* Copy 已使用的 Mutex
* 重入
* 死锁

## Lock/Unlock 不是成对出现

Lock/Unlock 没有成对出现，就意味着会出现死锁的情况，或者是因为 Unlock 一个未加锁的 Mutex 而导致 panic。

其中缺少Unlock的场景晁老师给出了三种：

1. 代码中有太多的 if-else 分支，可能在某个分支中漏写了 Unlock；
2. 在重构的时候把 Unlock 给删除了；
3. Unlock 误写成了 Lock。

乍一看每一种都很傻，这些情况下，锁被获取之后，就不会被释放了，这也就意味着，其它的 goroutine 永远都没机会获取到锁。其实很多大神也可能会犯这些错误。

缺少 Lock 的场景，这就很简单了，一般来说就是误操作删除了 Lock。

比如先前使用 Mutex 都是正常的，结果后来其他人重构代码的时候，由于对代码不熟悉，或者由于开发者的马虎，把 Lock 调用给删除了，或者注释掉了。比如下面的代码，mu.Lock() 一行代码被删除了，直接 Unlock 一个未加锁的 Mutex 会 panic：

1. 代码中有太多的 if-else 分支，可能在某个分支中漏写了 Unlock；
2. 在重构的时候把 Unlock 给删除了；
3. Unlock 误写成了 Lock。

```go
func foo() {
	var mu sync.Mutex
	defer mu.Unlock()
	fmt.Println("hello world!")
}
```

## Copy 已使用的 Mutex

这个点我印象很深，因为在看锁部分的源码是有留心过这部分的注释，就警告不要复制锁！

不能复制锁的原因也比较容易想到，因为截止目前的版本，锁已经是“带状态”的了：`state字段`​

如果你复制了一个已经加锁的Mutex给了新的变量，这个新的变量按理来说期望是零值。在这个时候，在并发环境下，执行它你就永远可能不知道它究竟是什么状态，因为要复制的mutex是由其它goroutine并发访问的，状态可能随时在变化。

如果不信邪，便一试究竟：

```go
type Counter struct {
	sync.Mutex
	Count int
}

func main() {
	var c Counter
	c.Lock()
	defer c.Unlock()
	c.Count++
	foo(c) // 复制锁
}

// 这里Counter的参数是通过复制的方式传入的
func foo(c Counter) {
	c.Lock()
	defer c.Unlock()
	fmt.Println("in foo")
}
```

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926050.png)​

第十一行的foo（c），这个时候传入的Couner实例已经是带状态的了

<u><span style="font-weight: bold;" data-type="strong">*如果解决这类死锁问题？</span>*</u>

其实IDE编译环境下其实就有提示了，让你注意了，比如goland环境：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312181926765.png)​

除此之外，go运行时还有死锁的检查机制`checkdead方法`​，它能够发现死锁的goroutine

但是有个重点：你肯定不想你的项目上线运行了才发现死锁问题吧！

使用 vet 工具，把检查写在 Makefile 文件中，在持续集成的时候跑一跑，这样可以及时发现问题，及时修复。我们可以使用 go vet 检查这个 Go 文件：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312141113613.png)​

使用这个工具就可以发现 Mutex 复制的问题，错误信息显示得很清楚，是在调用foo 函数的时候发生了 lock value 复制的情况，还告诉我们出问题的代码行数以及 copy lock 导致的错误

vet 工具是怎么发现 Mutex 复制使用问题的？

检查是通过`copylock分析器`​静态分析实现的。这个分析器会分析函数调用、range 遍历、复制、声明、函数返回值等位置，有没有锁的值 copy 的情景，以此来判断有没有问题。

只要是实现了 Locker 接口，就会被分析。

## 重入

一句话说明：重入锁允许同一个线程或 goroutine 多次获取同一把锁而不会造成死锁，但 Go 标准库中的 `sync.Mutex`​ 不支持重入。

关于重入的概念：

当一个线程获取锁时，如果没有其它线程拥有这个锁，那么，这个线程就成功获取到这个锁。之后，如果其它线程再请求这个锁，就会处于阻塞等待的状态。但是，如果拥有这把锁的线程再请求这把锁的话，不会阻塞，而是成功返回，所以叫可重入锁（有时候也叫做递归锁）。只要你拥有这把锁，你可以可着劲儿地调用，比如通过递归实现一些算法，调用者不会阻塞或者死锁。

但是问题就来了~

Mutex 不是可重入的锁！

为啥Mutex不可重入？

如果上述文章仔细读了肯定能知道：<span style="font-weight: bold;" data-type="strong">Mutex 的实现中没有记录哪个 goroutine 拥有这把锁。</span> 

理论上，任何 goroutine 都可以随意地 Unlock 这把锁，所以没办法计算重入条件。

重入锁示例：

```go
package main

import (
	"fmt"
	"sync"
)

func fooo(l sync.Locker) {
	fmt.Println("in foo")
	l.Lock()
	bar(l)
	l.Unlock()
}

func bar(l sync.Locker) {
	// 同一把锁，拥有这个锁的线程再次请求锁
	l.Lock()
	fmt.Println("in bar")
	l.Unlock()
}
func main() {
	l := &sync.Mutex{}
	fooo(l)
}

```

而我初步想到的例子，这种算不算可重入错误示例呢？

```go
package main

import (
	"fmt"
	"sync"
)

func fooo(l sync.Locker) {
	fmt.Println("in foo")
	l.Lock()
	l.Lock()
	l.Unlock()
}

func main() {
	l := &sync.Mutex{}
	fooo(l)
}

```

关键来了，go如何实现可重入锁？

这其中的关键就是记住哪个goroutine持有这个锁，这处自己可以很容易想到用一个id来标识goroutine，关键就是这个id是在整个阶段什么位置产生的

### 实现可重入锁方案一：goroutine id

获取goroutine id 又有两种方式：1.runtime.Stack 2.hacker

#### runtime.Stack

通过此方法获取栈帧信息，栈帧信息中包含goroutine id

```bash
goroutine 1 [running]:
main.main()
....../main.go:19 +0xb1
```

第一行格式为 goroutine xxx，其中 xxx 就是 goroutine id，只要解析出这个 id 即可。

解析的方式可以是：

```bash
func GoID() int {
	var buf [64]byte
	n := runtime.Stack(buf[:], false)
	// 得到id字符串
	idField := strings.Fields(strings.TrimPrefix(string(buf[:n]), "goroutine"))
	id, err := strconv.Atoi(idField)
	if err != nil {
		panic(fmt.Sprintf("cannot get goroutine id: %v", err))
	}
	return id
}
```

#### hacker

‍

‍

### 实现可重入锁方案二：token

‍

## 死锁

‍

‍

‍

## 著名Go项目issue案例

‍

‍

‍

‍

‍

‍

# Mutex扩展功能

‍

‍

‍

# 思考问题

<u><span style="font-weight: bold;" data-type="strong">如果 Mutex 已经被一个 goroutine 获取了锁，其它等待中的 goroutine 们只能一直等待。那么，等这个锁释放后，等待中的 goroutine 中哪一个会优先获取 Mutex 呢？</span></u>

首先一点：等待的goroutine们是以FIFO排队的

1）当Mutex处于正常模式时，若此时没有新goroutine与队头goroutine竞争，则队头goroutine获得。若有新goroutine竞争大概率新goroutine获得。

2）当队头goroutine竞争锁失败达到1ms后，它会将Mutex调整为饥饿模式。进入饥饿模式后，锁的所有权会直接从解锁goroutine移交给队头goroutine，此时新来的goroutine直接放入队尾。

3）当一个goroutine获取锁后，如果发现自己满足下列条件中的任何一个则将锁切换回正常模式：

     条件①：它是队列中最后一个；条件②：它等待锁的时间少于1ms

以上简略翻译自[https://golang.org/src/sync/mutex.go](https://golang.org/src/sync/mutex.go) 中注释Mutex fairness.

‍
