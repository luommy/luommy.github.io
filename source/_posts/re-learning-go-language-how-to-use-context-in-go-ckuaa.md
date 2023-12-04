---
title: 重学Go语言 | 如何在Go中使用Context
date: '2023-12-01 22:42:25'
updated: '2023-12-04 17:27:35'
excerpt: 重新认识Context
categories:
  - Golang
  - 重学系列
permalink: /post/re-learning-go-language-how-to-use-context-in-go-ckuaa.html
comments: true
toc: true
---

# 重学Go语言 | 如何在Go中使用Context

我们知道在开发Go应用程序时，尤其是网络应用程序，需要启动大量的`Goroutine`​来处理请求：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312041727947.png)​

不过`Goroutine`​被创建之后，除非执行后正常退出或者触发`panic`​退出，Go并没有提供在一个`Goroutine`​中关闭另一个`Goroutine`​的机制。

有没有一种方法，可以从一个`Goroutine`​中通知另一个`Goroutine`​退出执行呢？这时候就该`Context`​登场了！

# 什么是Context？

​`Context`​，中文叫做`上下文`​，Go语言在`1.7`​版本中新增的`context`​包中定义了`Context`​，`Context`​本质是一个接口，这个接口一共定义了四个方法：

```go
type Context interface {
	Deadline() (deadline time.Time, ok bool)
	Done() <-chan struct{}
	Err() error
	Value(key any) any
}
```

* ​`Dateline()`​：获取定时关闭的时间。
* ​`Done()`​：从一个`channel`​获取关闭的信号
* ​`Err()`​：获取错误信息。
* ​`Value()`​：根据`key`​从`Context`​取值

# Context的作用

为什么要使用`Context`​？或者说设计`Context`​的目的是什么？

想像一下这样的场景，当用户开启浏览器访问我们的`Web`​服务时，我们可能会开启多个`Goroutine`​来处理用户的请求，这些`Goroutine`​需要读取不同的资源，最终返回给用户，但如果用户在我们的Web服务还没处理完成就关闭浏览器，断开连接，而此时Web不知道用户已经关闭请求，仍然在处理并返回最终并不会被接收的数据。

​`Context`​设计的目的就是可以从上游的`Goroutine`​发送信息给下游的`Goroutine`​，回到处理用户请求的场景，当处理请求的`Goroutine`​发现用户断开连接时，通过`Context`​发送停止执行的信息，而下游的`Goroutine`​得到停止信号时就返回，避免资源的浪费。

概括起来，`Context`​的作用主要体现在两个方面：

* 在`Goroutine`​之间传递关闭信息，定时关闭，超时关闭，手动关闭。
* 在`Goroutine`​之间传递数据。

# Context的使用

下面我们来讲讲`Context`​的基本使用。

## 创建Context

任何上下文都是从一个空白的`Context`​开始的，创建一个空白的`Context`​有两种方式：

* 使用`context.Background()`​：

```go
ctx := context.Backgroud()
```

* 使用`contenxt.TODO()`​:

```go
ctx := context.TODO()
```

当然大部分时候，我们不需要自己创建一个空白的`Context`​，比如在处理`HTTP`​请求时：

```go
http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
 ctx := r.Context()
})
```

上面的代码中，可以从`http.Request`​中获取`Context`​实例，而`http.Request`​的实际上也是调用`context.Backgroud()`​:

```go
//net/http/request.go
func (r *Request) Context() context.Context {
 if r.ctx != nil {
  return r.ctx
 }
 return context.Background()
}
```

## 实例讲解

为了讲解`Context`​是如何在`Goroutine`​之间传递信号与数据，我们通过下面的案例进行说明：

这段代码是一个读取文件内容的函数ReadFile，它接受一个上下文对象ctx、文件名fileName和一个通道result用于返回读取到的文件内容。  
首先，通过os.Open函数打开文件，并在出现错误时返回。然后，创建一个空的totalResult切片，用于存储整个文件的内容。接下来，进入一个无限循环，在循环中使用select语句监听上下文的完成事件。如果上下文被取消或超时，将空切片[]byte{}发送到result通道，并返回函数。如果上下文没有完成，则继续执行循环体。在每次循环中，使用file.Read函数读取1024字节的数据到切片b中，并检查是否遇到了文件结束（EOF）错误。如果是，则将切片b的内容追加到totalResult切片中，并跳出循环。如果没有遇到文件结束错误，将切片b的内容追加到totalResult切片中，然后继续循环。当循环结束后，将完整的totalResult切片发送到result通道中，函数执行完毕。

```go

func ReadFile(ctx context.Context, fileName string, result chan<- []byte) {
	file, err := os.Open(fileName)
	if err != nil {
		return
	}
	totalResult := make([]byte, 0)
	for {
		select {
		case <-ctx.Done():
			result <- []byte{}
			return
		// default分支为空，这是为了确保在没有收到上下文完成事件时，循环不会阻塞在select语句上
		default:
		}
		b := make([]byte, 1024)
		//每次循环读取1024字节的数据到切片b中
		_, err := file.Read(b)
		if err == io.EOF {
			totalResult = append(totalResult, b...)
			break
		}
		totalResult = append(totalResult, b...)
	}
	result <- totalResult
}
```

在上面的程序中，我们调用`Context`​的`Done()`​方法，该方法会返回一个`Channel`​，而使用`select`​语句则可以让我们在处理业务的同时，监听上游`Goroutine`​是否有传递取消执行的信息。

## 手动取消：WithCancel设置Context传递停止信号

空白的`Context`​并不能发挥什么作用，要达到手动取消执行的目的，需要调用`context`​包下的`WithCancel`​函数进行封装，封装返回一个新的`context`​以及一个取消的句柄`cancel`​函数：

```go
ctx := context.Background()
ctx, cancel := context.WithCancel(ctx)
```

下面是完整的使用方法：将context对象传入到协程函数中，手动调用cancel进行取消

```go
package main

import (
 "context"
 "fmt"
 "io"
 "os"
 "time"
)

func main() {
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)
	result := make(chan []byte)     
	// 100ms后此协程被手动cancel取消    
	go ReadFile(ctx, "./test.tar", result) 
	time.Sleep(100 * time.Millisecond)
	cancel() 
	fmt.Println(<-result)
}
```

在上面的程序中，我们创建一个`Context`​之后，传给了`ReadFile`​函数，并且在暂停100毫秒后调用`cancel()`​函数，达到手动取消另一个`Goroutine`​的目的。

## 截止时间：WithDeadline给Context设置一个截止时间

除了手动取消，也可以调用`context`​包下的`WithDeadline()`​函数给`Context`​加一个截止时间，这样在某个时间点，`Context`​会自动发出取消信号：

```
afterTime := time.Now().Add(30 * time.Millisecond)
ctx, cancel := context.WithDeadline(context.Background(), afterTime)
```

下面是完整的示例：

```go
package main

import (
 "context"
 "fmt"
 "io"
 "os"
 "time"
)

func main() {
	afterTime := time.Now().Add(30 * time.Millisecond)
	ctx, cancel := context.WithDeadline(context.Background(), afterTime)
	defer cancel()
	result := make(chan []byte)
	go ReadFile(ctx, "./test.tar", result)
	fmt.Println(<-result)
}
```

## 超时取消：WithTimeout给Context一个超时时间

调用`context`​包下的`WithTimeout()`​函数可以为`Context`​加一个超时限制，这对于我们编写超时控制程序非常有帮助：

```
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Millisecond)
```

下面是完整的调用程序：

```go
package main

import (
 "context"
 "fmt"
 "io"
 "os"
 "time"
)

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Millisecond)
	defer cancel()
	result := make(chan []byte)
	go ReadFile(ctx, "./test.tar", result)
	fmt.Println(<-result)
}
```

## 传递数据：使用Context传递数据

调用`context`​包下的`WithValue()`​函数可以生成一个携带数据的`Context`​，这个机制方便我们跟踪一个处理流程中的`Goroutine`​：

```go
 ctx, cancel := context.WithValue(context.Background(), "testKey","testValue")
```

下游的`Goroutine`​就可以通过`Context`​的`Value()`​函数来获取上游传递下来的值了。

```golang
func MyGoroutine(ctx context.Context){
	ctx.Value("testKey")
}
```

思考：这里联想下业务场景

# 使用Context的几点建议

* ​`Context`​不要放在结构体中，需要以参数方式传递
* ​`Context`​作为函数参数时，一般放在第一位，作为函数的第一个参数
* 使用 `context.Background`​函数生成根节点的`Context`​
* ​`Context `​要传值必要的值，不要什么都传
* ​`Context`​ 是多协程安全的，可以在多个协程中使用

# 小结

至此，应该对`Context`​有所了解了吧，总的来说，通过`Context`​可以做到：

* WithCancel手动控制下游`Goroutine`​取消执行。
* WithDeadline定时控制下游`Goroutine`​取消执行。
* WithTimeout超时控制下游`Goroutine`​取消执行。
* WithValue传递数据给下游的`Goroutine`​。
