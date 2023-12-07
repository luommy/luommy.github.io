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

> 多个 goroutine 并发更新同一个资源，像计数器；同时更新用户的账户信息；秒杀系统；往同一个 buffer 中并发写入数据等等。如果没有互斥控制，就会出现一些异常情况，比如计数器的计数不准确、用户的账户可能出现透支、秒杀系统出现超卖、buffer 中的数据混乱，等等，后果都很严重。

这些问题怎么解决呢？对，用互斥锁，那在 Go 语言里，就是 Mutex。

并发编程中涉及一个概念，叫做临界区。临界区就是一个被共享的资源，或者说是一个整体的一组共享资源，比如对数据库的访问、对某一个共享数据结构的操作、对一个 I/O 设备的使用、对一个连接池中的连接的调用，等等。为防止访问、操作错误，使用互斥锁，限定临界区只能同时由一个线程持有。

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312071408008.png)​

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

# 如果不同Mutex锁

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

# 并发问题排查工具race detector以及原理

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

# 使用mutex

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

# mutex常用操作

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

‍
