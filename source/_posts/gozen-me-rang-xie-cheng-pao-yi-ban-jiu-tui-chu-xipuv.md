---
title: Go怎么让协程跑一半就退出？
date: '2023-12-12 11:35:58'
updated: '2023-12-12 14:22:00'
excerpt: 认识goexit、理解GMP实例、函数栈、底层调度
tags:
  - golang
categories:
  - Golang
  - 核心底层
permalink: /post/gozen-me-rang-xie-cheng-pao-yi-ban-jiu-tui-chu-xipuv.html
comments: true
toc: true
---

# Go怎么让协程跑一半就退出？

> go中runtime.Goexit()可以让跑一半的协程退出？
>
> 我的第一反应是创建了一个`context.Context`​对象，并通过`context.WithCancel`​函数创建一个可以取消的上下文。然后，在协程中使用`select`​语句监听`ctx.Done()`​通道，当接收到退出信号时，协程会执行退出操作。
>
> 参考小白了解到了一些不一样的东西

我的：

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	go func(ctx context.Context) {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("协程收到退出信号，退出")
				return
			default:
				// 执行协程任务
				fmt.Println("正在执行任务...")
				time.Sleep(time.Second)
			}
		}
	}(ctx)

	// 等待一段时间后取消协程
	time.Sleep(3 * time.Second)
	cancel()

	// 等待协程退出
	time.Sleep(time.Second)
	fmt.Println("主线程退出")
}

```

我们平时创建一个协程，跑一段逻辑，代码大概长这样。

```go
package main
 
import (
    "fmt"
    "time"
)
func Foo() {
    fmt.Println("打印1")
    defer fmt.Println("打印2")
    fmt.Println("打印3")
}
 
func main() {
    go  Foo()
    fmt.Println("打印4")
    time.Sleep(1000*time.Second)
}
 
// 这段代码，正常运行会有下面的结果
打印4
打印1
打印3
打印2
```

<span style="font-weight: bold;" data-type="strong">注意</span>这上面"​<span style="font-weight: bold;" data-type="strong">打印2</span>​"是在`defer`​中的，所以会在函数结束前打印。因此后置于"​<span style="font-weight: bold;" data-type="strong">打印3</span>​"。

那么今天的问题是，如何让`Foo()`​函数<span style="font-weight: bold;" data-type="strong">跑一半就结束</span>，比如说跑到<span style="font-weight: bold;" data-type="strong">打印2</span>，就退出协程（不让defer中的“3”打印）。输出如下结果

```
打印4
打印1
打印2
```

如何实现？

答：在"打印2"后面插入一个 `runtime.Goexit()`​​， 协程就会直接结束。并且结束前还能执行到`defer`​​里的​<span style="font-weight: bold;" data-type="strong">打印2</span>​。

```go
package main
 
import (
    "fmt"
    "runtime"
    "time"
)
func Foo() {
    fmt.Println("打印1")
    defer fmt.Println("打印2")
    runtime.Goexit() // 加入这行代码 可实现打印2结束退出
    fmt.Println("打印3")
}
 
func main() {
    go  Foo()
    fmt.Println("打印4")
    time.Sleep(1000*time.Second)
}
 
 
// 输出结果
打印4
打印1
打印2
```

可以看到<span style="font-weight: bold;" data-type="strong">打印3</span>这一行没出现了，协程确实提前结束了。

# 一图梗概

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312121420729.png)​

# runtime.Goexit()是什么？

看一下内部实现。

```go
func Goexit() {
    // 以下函数省略一些逻辑...
    gp := getg() 
    for {
    // 获取defer并执行
        d := gp._defer
        reflectcall(nil, unsafe.Pointer(d.fn), deferArgs(d), uint32(d.siz), uint32(d.siz))
    }
    goexit1()
}
 
func goexit1() {
    mcall(goexit0)
}
```

从代码上看，`runtime.Goexit()`​会先执行一下`defer`​里的方法，这里就解释了开头的代码里为什么<span style="font-weight: bold;" data-type="strong">在defer里的打印2</span>能正常输出。

然后代码再执行`goexit1`​。本质就是对`goexit0`​的简单封装。

我们可以把代码继续跟下去，看看`goexit0`​做了什么。

```go
// goexit continuation on g0.
func goexit0(gp *g) {
  // 获取当前的 goroutine
    _g_ := getg()
    // 将当前goroutine的状态置为 _Gdead
    casgstatus(gp, _Grunning, _Gdead)
  // 全局协程数减一
    if isSystemGoroutine(gp, false) {
        atomic.Xadd(&sched.ngsys, -1)
    }
  
  // 省略各种清空逻辑...
 
  // 把g从m上摘下来。
  dropg()
 
 
    // 把这个g放回到p的本地协程队列里，放不下放全局协程队列。
    gfput(_g_.m.p.ptr(), gp)
 
  // 重新调度，拿下一个可运行的协程出来跑
    schedule()
}
 
```

这段代码，信息密度比较大。

很多名词可能让人一脸懵。

简单描述下，Go语言里有个<span style="font-weight: bold;" data-type="strong">GMP模型</span>的说法，`M`​是内核线程，`G`​也就是我们平时用的协程`goroutine`​，`P`​会在`G和M之间`​做工具人，负责<span style="font-weight: bold;" data-type="strong">调度</span>​`G`​到`M`​上运行。

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312121420761.png "GMP模型")  

既然是​<span style="font-weight: bold;" data-type="strong">调度</span>​，也就是说不是每个`G`​都能一直处于运行状态，等G不能运行时，就把它存起来，再<span style="font-weight: bold;" data-type="strong">调度</span>下一个能运行的G过来运行。

暂时不能运行的G，P上会有个<span style="font-weight: bold;" data-type="strong">本地队列</span>去存放这些这些G，P的本地队列存不下的话，还有个全局队列，干的事情也类似。

了解这个背景后，再回到 `goexit0`​ 方法看看，做的事情就是将当前的协程G置为`_Gdead`​状态（说白了这个runtime.Goexit()就是让这个线程去死），即：然后把它从M上摘下来，尝试放回到P的本地队列中。然后重新调度一波，获取另一个能跑的G（看看其它有没有能跑的协程），拿出来跑。

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312121420477.gif "goexit函数：P调度可用的G到M上")  

所以简单总结一下：<span style="font-weight: bold;" data-type="strong">只要执行 goexit 这个函数，当前协程就会退出，同时还能调度下一个可执行的协程出来跑。</span>

这就是为什么`runtime.Goexit()`​能让协程只执行一半就结束了。

# goexit的用途

我理解是：堆栈底层 "御用" 退出方法

正经人谁会没事跑一半协程就结束呢？所以`goexit`​的<span style="font-weight: bold;" data-type="strong">真实用途</span>是啥？

有个​<span style="font-weight: bold;" data-type="strong">小细节</span>​，不知道大家平时debug的时候有没有关注过。

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312121420656.jpg "小细节")​

为了说明问题，这里先给出一段代码。

```go
package main
 
import (
    "fmt"
    "time"
)
func Foo() {
    fmt.Println("打印1")
}
 
func main() {
    go  Foo()
    fmt.Println("打印3")
    time.Sleep(1000*time.Second)
}
```

这是一段非常简单的代码，输出什么完全不重要。通过`go`​关键字启动了一个`goroutine`​执行`Foo()`​，里面打印一下就结束，主协程`sleep`​很长时间，只为​<span style="font-weight: bold;" data-type="strong">死等</span>​。

这里我们新启动的协程里，在`Foo()`​函数内随便打个断点。然后`debug`​一下。

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312121420334.png "null")​

会发现，这个协程的堆栈底部是从`runtime.goexit()`​里开始启动的。

如果大家平时有注意观察，会发现，​<span style="font-weight: bold;" data-type="strong">其实所有的堆栈底部，都是从这个函数开始的</span>​。我们继续跟跟代码。

# goexit是什么？

从上面的`debug`​堆栈里点进去会发现，这是个汇编函数，可以看出调用的是`runtime`​包内的 `goexit1()`​ 函数。

```go
// The top-most function running on a goroutine
// returns to goexit+PCQuantum.
TEXT runtime·goexit(SB),NOSPLIT,$0-0
    BYTE    $0x90    // NOP
    CALL    runtime·goexit1(SB)    // does not return
    // traceback from goexit1 must hit code range of goexit
    BYTE    $0x90    // NOP
```

于是跟到了`pruntime/proc.go`​里的代码中。

```go
// 省略部分代码
func goexit1() {
    mcall(goexit0)
}
```

是不是很熟悉，这不就是我们开头讲`runtime.Goexit()`​里内部执行的`goexit0`​吗。

# 为什么每个堆栈底部都是这个方法？

因为只要创建协程就有`newproc1`​这个方法，它获取当前协程G所在的调度器P，然后创建一个新G，并在栈底插入一个goexit

我们首先需要知道的是，函数栈的执行过程，是先进后出。

假设我们有以下代码

```go
func main() {
    B()
}
 
func B() {
    A()
}
 
func A() {
 
}
```

上面的代码是main运行B函数，B函数再运行A函数，代码执行时就跟下面的动图那样。

​![动态演示](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312121420259.gif "函数堆栈执行顺序")  

这个是先进后出的过程，也就是我们常说的函数栈，执行完<span style="font-weight: bold;" data-type="strong">子函数A()</span> 后，就会回到<span style="font-weight: bold;" data-type="strong">父函数B()</span> 中，执行完<span style="font-weight: bold;" data-type="strong">B()后</span>，最后就会回到<span style="font-weight: bold;" data-type="strong">main()</span> 。这里的栈底是`main()`​，如果在<span style="font-weight: bold;" data-type="strong">栈底</span>插入的是 `goexit`​ 的话，那么当程序执行结束的时候就都能执行到`goexit`​里去。

结合前面讲过的内容，我们就能知道，此时栈底的`goexit`​，会在协程内的业务代码跑完后被执行到，从而实现协程退出，并调度下一个<span style="font-weight: bold;" data-type="strong">可执行的G</span>来运行。

 <u>*——那么问题又来了，栈底插入*</u>​`<u>*goexit*</u>`​<u>*这件事是谁做的，什么时候做的？*</u>

直接说答案，这个在`runtime/proc.go`​里有个`newproc1`​方法，只要是<span style="font-weight: bold;" data-type="strong">创建协程</span>都会用到这个方法（main是主协程也不例外）。

里面有个地方是这么写的：

```go
func newproc1(fn *funcval, argp unsafe.Pointer, narg int32, callergp *g, callerpc uintptr) {
    // 获取当前g
  _g_ := getg()
    // 获取当前g所在的p
    _p_ := _g_.m.p.ptr()
  // 创建一个新 goroutine
    newg := gfget(_p_)
 
    // 底部插入goexit
    newg.sched.pc = funcPC(goexit) + sys.PCQuantum 
    newg.sched.g = guintptr(unsafe.Pointer(newg))
    // 把新创建的g放到p中
    runqput(_p_, newg, true)
 
    // ...
}
 
```

主要的逻辑是获取当前协程G所在的调度器P，然后创建一个新G，并在栈底插入一个goexit。

所以我们每次debug的时候，就都能看到函数栈底部有个goexit函数。

# main函数也是个协程，栈底也是goexit？

关于main函数栈底是不是也有个`goexit`​，我们对下面代码断点看下。直接得出结果。

	​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312121420770.png "null")​

main函数栈底也是`goexit()`​

从 `asm_amd64.s`​可以看到Go程序启动的流程，这里提到的 `runtime·mainPC`​ 其实就是 `runtime.main`​.

```
    // create a new goroutine to start program
    MOVQ    $runtime·mainPC(SB), AX        // 也就是runtime.main
    PUSHQ    AX
    PUSHQ    $0            // arg size
    CALL    runtime·newproc(SB)
```

通过`runtime·newproc`​创建`runtime.main`​协程，然后在`runtime.main`​里会启动`main.main`​函数，这个就是我们平时写的那个main函数了。

```go
// runtime/proc.go
func main() {
    // 省略大量代码
    fn := main_main // 其实就是我们的main函数入口
    fn() 
}
 
//go:linkname main_main main.main
func main_main()
```

结论是，<span style="font-weight: bold;" data-type="strong">其实main函数也是由newproc创建的，只要通过newproc创建的goroutine，栈底就会有一个goexit。</span>

# os.Exit()和runtime.Goexit()有什么区别

最后再回到开头的问题，实现一下首尾呼应。

开头的面试题，除了`runtime.Goexit()`​，是不是还可以改为用`os.Exit()`​？

同样都是带有"退出"的含义，两者退出的<span style="font-weight: bold;" data-type="strong">对象</span>不同。`os.Exit()`​ 指的是整个<span style="font-weight: bold;" data-type="strong">进程</span>退出；而`runtime.Goexit()`​指的是<span style="font-weight: bold;" data-type="strong">协程</span>退出。

可想而知，改用`os.Exit()`​ 这种情况下，defer里的内容就不会被执行到了。

```go
package main
 
import (
    "fmt"
    "os"
    "time"
)
func Foo() {
    fmt.Println("打印1")
    defer fmt.Println("打印2")
    os.Exit(0)
    fmt.Println("打印3")
}
 
func main() {
    go  Foo()
    fmt.Println("打印4")
    time.Sleep(1000*time.Second)
}
 
// 输出结果
打印4
打印1
 
```

# 总结

* 通过 `runtime.Goexit()`​可以做到提前结束协程，且结束前还能执行到defer的内容
* ​`runtime.Goexit()`​其实是对`goexit0`​的封装，只要执行 `goexit0`​这个函数，当前协程就会退出，同时还能调度下一个可执行的协程出来跑。

* 通过`newproc`​可以创建出新的`goroutine`​，它会在函数栈底部插入一个`goexit`​
* ​`os.Exit()`​ 指的是整个<span style="font-weight: bold;" data-type="strong">进程</span>退出；而`runtime.Goexit()`​指的是<span style="font-weight: bold;" data-type="strong">协程</span>退出。两者含义有区别！

‍
