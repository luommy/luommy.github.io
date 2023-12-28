---
title: 利用GODEBUG更直观地理解 Go 调度过程
date: '2023-12-28 10:40:09'
updated: '2023-12-28 10:40:10'
excerpt: 利用GODEBUG工具验证GMP的调度实例
tags:
  - golang
categories:
  - Golang
  - 核心底层
permalink: >-
  /post/using-godebug-more-intuitively-to-understand-the-go-scheduling-process-z1bawyb.html
comments: true
toc: true
---



参考：Golang技术分享

调度系统，其中最核心的就是 GMP 的设计，欲深入理解 Go 语言设计的读者都应该看过这些知识。但是，在通过相关博客或者源码学习时，如果不能和实际的代码进行结合，在理解上或许不够深刻。

本文介绍一种方式，即使用 GODEBUG工具 ，通过实际运行代码来直观地查看 Go 运行时的调度过程。

# 调度简述

首先，先通过程序某时刻的调度快照示意图，通过解析快照状态来简单回顾一下 Go 调度系统。

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312281040259.png)​

如上图所示，设定了 GOMAXPROCS=2，即 2 个处理器。

当前时刻， P0 和 P1 上正分别挂载着 OS 线程 M1 与 M4，其上分别执行着 G8 和 G17 的代码。P0 的 LRQ（Local Run Queue，本地运行队列）有 3 个 G 在排队等待，而 P1 的 LRQ 已无等待的 G；GRQ （Global Run Queue，全局运行队列）中有 5 个 G 。

网络轮询器 Net Poller 上有一个陷入异步网络调用的 G9；M2 由于 G11 的某种同步系统调用而阻塞；M3 处于空闲状态，时刻准备着当 M1 或 M4 被阻塞时而派上用场。

由于 P1 的 LRQ 已无等待的 G，当 G17 被调度时，它将进行 Wrok Stealing （任务窃取），其窃取源来自于其他处理器 P 的 LRQ（这里是 P0 的 LRQ）、GRQ 和 Net Poller，具体规则见`runtime.schedule()`​函数。

# GODEBUG 工具

启用 GODEBUG 工具非常简单，只需要设置环境变量 GODEBUG 即可。它可以让 Go 程序在运行过程中输出调试信息，能够根据参数配置直观地看到调度器或垃圾回收等详细信息。

GODEBUG 的详细描述介绍可见源码`runtime/extern.go`​文件。

本文我们关心的调试内容是调度器，因此我们只使用 GODEBUG 的两个参数 schedtrace 与 scheddetail。

* schedtrace=n：设置运行时在每 n 毫秒输出一行调度器的概要信息。
* scheddetail: 输出更详细的调度信息。

## 示例代码

我们使用的示例如下

​​代码在下面。

```go
package main

import "sync"

var wg sync.WaitGroup

/**
   @Title main
   @Description  启动 20 个 CPU 密集型的 G 任务，它们受到 WaitGroup 的限制。当所有计算任务的 G 完成了各自的累加工作，程序才会结束执行。
   @Author luommy 2023-11-28 14:27:47
**/

// GODEBUG 的两个参数 schedtrace 与 scheddetail。
// schedtrace=n：设置运行时在每 n 毫秒输出一行调度器的概要信息。
// scheddetail: 输出更详细的调度信息。

func main() {
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go work(&wg)
	}
	wg.Wait()
}

func work(wg *sync.WaitGroup) {
	var count int
	for i := 0; i < 1e10; i++ {
		count++
	}
	wg.Done()
}
```

代码比较简单，我们启动 20 个 CPU 密集型的 G 任务，它们受到 WaitGroup 的限制。当所有计算任务的 G 完成了各自的累加工作，程序才会结束执行。

## schedtrace 调度概要输出

下面，我们设定 GODEBUG=schedtrace=1000，这意味着 1s 输出一次程序的调度概要情况。

```
 $ go build -o gmp1demo gmp1.go
 $ GOMAXPROCS=4 GODEBUG=schedtrace=1000 ./gmp1demo
SCHED 0ms: gomaxprocs=4 idleprocs=3 threads=2 spinningthreads=0 idlethreads=0 runqueue=0 [0 0 0 0]
SCHED 1003ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=14 [1 0 1 0]
SCHED 2012ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=10 [2 1 2 1]
SCHED 3018ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=12 [0 0 0 4]
SCHED 4029ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=15 [0 0 1 0]
SCHED 5031ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=9 [1 2 2 2]
SCHED 6035ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=15 [0 1 0 0]
SCHED 7044ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=8 [1 2 3 2]
SCHED 8054ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=12 [0 0 4 0]
SCHED 9055ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=6 [3 2 3 2]
SCHED 10063ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=11 [1 2 1 1]
SCHED 11072ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=6 [3 2 3 2]
...
```

其中，

* ​`SCHED`​：代表程序启动到输出当前行时的运行时间，这个输出间隔受到 schedtrace 值影响。
* ​`gomaxprocs`​：GOMAXPROCS 值，这里我们设定了其为 4。
* ​`idleprocs`​：空闲的 P 数量。
* ​`threads`​：运行时管理的线程数。
* ​`spinningthreads`​：自旋线程，处于”自旋“状态的线程数（避免频繁的线程创建与销毁）。
* ​`idlethreads`​：空闲线程数。
* ​`runqueue`​：全局队列 GRQ 中的 G 数量。
* [2 1 2 1]：代表 4 个 P 的本地队列 LRQ 中 G 数量分别是 2、1、2、1 。

我这里是在Windows下powershell中执行的：

```shell
$env:GOMAXPROCS = 4
$env:GODEBUG = "schedtrace=1000,scheddetail=1"
.\gmp1demo
```

目录是这样的：

```shell
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        2023/11/28     14:33            677 gmp1.go
-a----        2023/11/28     14:31        1178112 gmp1demo
```

注意：

在 Go 中，`GOMAXPROCS`​ 的默认设置是根据计算机的 CPU 核心数自动确定的。如果你显式地设置了 `GOMAXPROCS`​ 环境变量或调用了 `runtime.GOMAXPROCS`​ 函数来更改了默认设置，你可以通过以下方法将其恢复到默认值：

1. 清除 `GOMAXPROCS`​ 环境变量：你可以使用操作系统提供的命令来清除 `GOMAXPROCS`​ 环境变量。具体的方法取决于你使用的操作系统和终端。
2. 重启程序：如果你在程序中通过调用 `runtime.GOMAXPROCS`​ 来更改了默认设置，可以简单地重新启动程序，让它再次使用默认的 `GOMAXPROCS`​ 值。
3. 调用 `runtime.GOMAXPROCS`​ 函数并传递参数为 0：你可以在程序中调用 `runtime.GOMAXPROCS(0)`​ 来将 `GOMAXPROCS`​ 设置为默认值。这将根据计算机的 CPU 核心数重新设置 `GOMAXPROCS`​。

查看电脑执行的CPU核心数即默认情况下的<span style="font-weight: bold;" data-type="strong">GOMAXPROCS</span>：

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	defaultMaxProcs := runtime.NumCPU()
	fmt.Println("默认的 GOMAXPROCS 值为：", defaultMaxProcs)
}

```

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312280903146.png)​

公司这台机器是12。

使用命令重置，将GOMAXPROCS恢复成默认：

<span style="font-weight: bold;" data-type="strong">Windows</span>

​`  $env:GOMAXPROCS=""`​

<span style="font-weight: bold;" data-type="strong">Linux</span>

​`  unset GOMAXPROCS`​

查看GOMAXPROCS：

如果想要在命令行中查看 `GOMAXPROCS`​ 的值，可以使用以下命令：

在 Windows 系统的命令提示符（cmd）中：

```
echo %GOMAXPROCS%
```

在类 Unix 系统的终端中：

```
echo $GOMAXPROCS
```

执行这些命令后，会在命令行中显示当前的 `GOMAXPROCS`​​ 值。

如果没有那就是未设置或者设置为空，则直接由CPU核心决定的

​`GOMAXPROCS`​​ 是 Go 语言中一个用于控制并发执行的环境变量。它决定了同时运行的 Goroutine 的最大数量。并不是设置 `GOMAXPROCS`​​ 的值越大越好。过高的值可能会导致过多的 Goroutine 竞争 CPU 资源，从而降低性能。

## scheddetail 调度详细输出

当我们想要查看更详细的调度信息时，需要增加 scheddetail 参数。

```
$ GOMAXPROCS=4 GODEBUG=schedtrace=1000,scheddetail=1 ./demo
SCHED 0ms: gomaxprocs=4 idleprocs=2 threads=3 spinningthreads=1 idlethreads=0 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=1 schedtick=0 syscalltick=0 m=0 runqsize=0 gfreecnt=0 timerslen=0
  P1: status=1 schedtick=0 syscalltick=0 m=2 runqsize=0 gfreecnt=0 timerslen=0
  P2: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
  P3: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0 timerslen=0
  M2: p=1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=2 dying=0 spinning=false blocked=false lockedg=-1
  M1: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=2 dying=0 spinning=false blocked=false lockedg=-1
  M0: p=0 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=1
  G1: status=1() m=-1 lockedm=0
  G2: status=1() m=-1 lockedm=-1
  G3: status=1() m=-1 lockedm=-1
SCHED 1000ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=15 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=1 schedtick=45 syscalltick=0 m=2 runqsize=0 gfreecnt=0 timerslen=0
  P1: status=1 schedtick=46 syscalltick=0 m=3 runqsize=0 gfreecnt=0 timerslen=0
  P2: status=1 schedtick=45 syscalltick=0 m=4 runqsize=1 gfreecnt=0 timerslen=0
  P3: status=1 schedtick=45 syscalltick=0 m=0 runqsize=0 gfreecnt=0 timerslen=0
  M4: p=2 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
  M3: p=1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
  M2: p=0 curg=40 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  M1: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=2 dying=0 spinning=false blocked=false lockedg=-1
  M0: p=3 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
  G1: status=4(semacquire) m=-1 lockedm=-1
  G2: status=4(force gc (idle)) m=-1 lockedm=-1
  G3: status=4(GC sweep wait) m=-1 lockedm=-1
  G17: status=4(GC scavenge wait) m=-1 lockedm=-1
  G33: status=1() m=-1 lockedm=-1
  G34: status=1() m=-1 lockedm=-1
  G35: status=1() m=-1 lockedm=-1
  G36: status=1() m=-1 lockedm=-1
  G37: status=1() m=-1 lockedm=-1
  G38: status=1() m=-1 lockedm=-1
  G39: status=1() m=-1 lockedm=-1
  G40: status=2() m=2 lockedm=-1
  G41: status=1() m=-1 lockedm=-1
  G42: status=1() m=-1 lockedm=-1
  G43: status=1() m=-1 lockedm=-1
  G44: status=1() m=-1 lockedm=-1
  G45: status=1() m=-1 lockedm=-1
  G46: status=1() m=-1 lockedm=-1
  G47: status=1() m=-1 lockedm=-1
  G48: status=1() m=-1 lockedm=-1
  G49: status=1() m=-1 lockedm=-1
  G50: status=1() m=-1 lockedm=-1
  G51: status=1() m=-1 lockedm=-1
  G52: status=1() m=-1 lockedm=-1
...
```

当增加了 scheddetail 参数后，其输出信息不仅包含了 SCHED 的一行概览信息，还增加了 GPM 三个实体状况的详细描述。

### P

* status：P 的运行状态，其详细分类与描述可查看源码 `runtime/runtime2.go`​ 的代码。

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312280916067.png)​

* schedtick：随着每次调度行为累加，代表 P 的调度次数。
* syscalltick：随着每次系统调用行为累加，代表 P 的系统调用次数。
* m: 绑定的 M 编号，例如在 SCHED 1000ms 时，P0 的 m=2，而 M2 的 p=0。
* runqsize：LRQ 的 G 数量。
* gfreecnt：状态为 Gdead 的数量，即 status = 6。
* timerslen：timer 的数量。

### M

* p：绑定的 P 编号。
* curg：当前正在 M 上执行代码的 G 编号。
* mallocing：是否正存在分配内存操作。
* throwing：是否有抛出异常。
* preemptoff：如果 preemptoff != ""，则保持 curg 在这个 M 上运行。
* locks：M 的 locks 数量。
* dying：M 的 dying 值，其存在 0、1、2 和其他值四种处理情况。
* spinning：是否处于自选状态。
* blocked：是否处于阻塞状态。
* lockedg：与 G 的 lockedm 相对应，它们的类型是 uintptr，记录不被垃圾收集器跟踪的 M 与绕过写屏障的 G。

### G

* status: 同 P 的状态类似，其详细分类与描述同样可查看源码 `runtime/runtime2.go`​ 的代码；if status==Gwaiting ，即 status 的值为 4 时，其括号内还会输出具体的等待原因。
* m：绑定的 M 编号，如果其值为 -1，代表无绑定。
* lockedm：与 M 中的 lockedg 对应。

## 可视化调度快照

明白了上述各项指标的含义之后。为了绘图简单，我们将 GOMAXPROCS 设定为2，并选取第 1 秒的输出内容进行可视化分析。

```
$ GOMAXPROCS=2 GODEBUG=schedtrace=1000,scheddetail=1 ./demo
...
SCHED 1004ms: gomaxprocs=2 idleprocs=0 threads=4 spinningthreads=0 idlethreads=1 runqueue=10 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=1 schedtick=45 syscalltick=0 m=3 runqsize=4 gfreecnt=0 timerslen=0
  P1: status=1 schedtick=48 syscalltick=0 m=0 runqsize=4 gfreecnt=0 timerslen=0
  M3: p=0 curg=24 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  M2: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M1: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=2 dying=0 spinning=false blocked=false lockedg=-1
  M0: p=1 curg=34 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  G1: status=4(semacquire) m=-1 lockedm=-1
  G2: status=4(force gc (idle)) m=-1 lockedm=-1
  G3: status=4(GC sweep wait) m=-1 lockedm=-1
  G4: status=4(GC scavenge wait) m=-1 lockedm=-1
  G17: status=1() m=-1 lockedm=-1
  G18: status=1() m=-1 lockedm=-1
  G19: status=1() m=-1 lockedm=-1
  G20: status=1() m=-1 lockedm=-1
  G21: status=1() m=-1 lockedm=-1
  G22: status=1() m=-1 lockedm=-1
  G23: status=1() m=-1 lockedm=-1
  G24: status=2() m=3 lockedm=-1
  G25: status=1() m=-1 lockedm=-1
  G26: status=1() m=-1 lockedm=-1
  G27: status=1() m=-1 lockedm=-1
  G28: status=1() m=-1 lockedm=-1
  G29: status=1() m=-1 lockedm=-1
  G30: status=1() m=-1 lockedm=-1
  G31: status=1() m=-1 lockedm=-1
  G32: status=1() m=-1 lockedm=-1
  G33: status=1() m=-1 lockedm=-1
  G34: status=2() m=0 lockedm=-1
  G35: status=1() m=-1 lockedm=-1
  G36: status=1() m=-1 lockedm=-1
...
```

该时刻的调度情况快照图示如下

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312280904124.png)​

我们可以根据详细信息获取到 P、M、G 的具体运行情况。但需要注意的是，有一点我们不能确定，就是各个 P 的 LRQ 与 GRQ 是哪些具体的 G 在队列中等待，但这并不妨碍大局（因此，上图中的 LRQ 和 GRQ 可能并不是实际的 G 编号）。

# 总结

本文介绍了通过增加环境变量 GODEBUG ,可以在不做任何代码改变或增加额外的插件情况下，方便地查看 Go 程序的调度情况。若想更直观地理解 Go 语言的 GMP 和调度系统过程，唯有一试 。
