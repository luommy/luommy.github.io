---
title: 练习两年半释疑
date: '2023-08-28 22:42:24'
updated: '2023-08-29 20:32:52'
excerpt: 练习时长两年半了，对新手很是困扰
tags:
  - golang
permalink: /post/practice-for-two-and-a-half-years-to-interpret-questions-z1mtq7y.html
comments: true
toc: true
---
# 练习两年半释疑

2023.8

时隔近两年半，书签问题与疑惑整理

‍

# go只有值传递而没有引用传递

通过一个例子来进一步理解Go语言只有值传递而没有引用传递的概念。

首先，我们来看一个简单的例子，这个例子中我们将一个整数变量传递给一个函数：

```go
func modify(x int) {
    x = 100
}

func main() {
    x := 1
    modify(x)
    fmt.Println(x) // 输出 1
}
```

在这个例子中，`modify`​ 函数试图修改它的参数 `x`​。但是，因为Go只有值传递，所以 `modify`​ 函数实际上接收的是 `x`​ 的副本，而不是 `x`​ 本身。因此，`modify`​ 函数的修改不会影响到 `main`​ 函数中的 `x`​。

现在，让我们看一个稍微复杂一点的例子，这个例子中我们将一个切片传递给一个函数：

```go
func modify(s []int) {
    s[0] = 100
}

func main() {
    s := []int{1, 2, 3}
    modify(s)
    fmt.Println(s) // 输出 [100 2 3]
}
```

在这个例子中，`modify`​ 函数修改了它的参数 `s`​ 的第一个元素。尽管 `s`​ 是通过值传递给 `modify`​ 的，但是这个值实际上是对底层数组的引用，因此 `modify`​ 的修改对 `main`​ 函数是可见的。

这两个例子展示了Go语言只有值传递的特性。在第一个例子中，我们看到了值类型（如整数）的值传递行为。在第二个例子中，我们看到了引用类型（如切片）的值传递行为。尽管在表面上看起来像是引用传递，但实际上仍然是值传递，只不过传递的值是一个引用。

# go的引用数据类型有哪些

Go语言中的引用类型有以下几种：

1. **切片（Slice）**：切片是对数组的抽象。Go数组的长度不可改变，在特定的场景中这样的集合就不太适用，Go中提供了一种灵活，功能强大的内置类型切片("动态数组")，与数组相比切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大。
2. **映射（Map）**：映射是一种无序的基于key-value的数据结构，Go语言中的映射关系使用hash实现。
3. **通道（Channel）**：通道（channel）是用来传递数据的一个数据结构。通道可用于两个goroutine之间通过传递一个指定类型的值来同步运行和通讯。操作符`<-`​用于指定通道的方向，发送或接收。如果未指定方向，则为双向通道。
4. **接口（Interface）**：接口是一种类型，它定义了一套方法（方法集），但是这些方法不包含（实现）代码：它们没有被实现（它们是抽象的）。接口里不能包含变量。
5. **指针（Pointer）**：Go语言中的指针类似于C和C++中的指针，但是Go语言中的指针不能进行偏移和运算，是安全指针。Go语言中的值类型都有对应的指针类型，形式为`*数据类型`​，例如`*int`​、`*string`​等。

以上就是Go语言中的引用类型，它们都可以被用来创建复杂的数据结构。

# go env各参数的理解与go get下载的包

这些环境变量是Go编程语言的配置参数，下面是它们的含义：

1. ​`GO111MODULE`​：这是Go的模块模式开关，当设置为`on`​时，表示启用模块模式，无论在GOPATH目录下还是非GOPATH目录下都会使用模块模式。
2. ​`GOARCH`​：目标平台的处理器架构，例如amd64、386、arm、ppc64等。
3. ​`GOBIN`​：编译后二进制的安装位置。
4. ​`GOCACHE`​：Go编译缓存的位置。
5. ​`GOENV`​：Go环境配置文件的位置。
6. ​`GOEXE`​：目标平台的可执行文件扩展名，例如在Windows上是`.exe`​。
7. ​`GOEXPERIMENT`​：用于启用或禁用实验性的编译器或链接器功能。
8. ​`GOFLAGS`​：传递给go工具的全局标志。
9. ​`GOHOSTARCH`​：Go工具运行的处理器架构。
10. ​`GOHOSTOS`​：Go工具运行的操作系统。
11. ​`GOINSECURE`​：指定那些不需要通过HTTPS和SUM数据库校验的代码仓库。
12. ​`GOMODCACHE`​：模块的下载缓存目录。
13. ​`GONOPROXY`​：指定那些不需要通过代理的代码仓库。
14. ​`GONOSUMDB`​：指定那些不需要通过SUM数据库校验的代码仓库。
15. ​`GOOS`​：目标平台的操作系统，例如linux、windows、darwin等。
16. ​`GOPATH`​：Go语言的工作目录，通常有三个子目录：bin、pkg、src。
17. ​`GOPRIVATE`​：指定那些不需要通过代理和不需要通过SUM数据库校验的代码仓库。
18. ​`GOPROXY`​：Go模块的代理服务器。
19. ​`GOROOT`​：Go语言的安装路径。
20. ​`GOSUMDB`​：Go模块的校验和数据库。
21. ​`GOTMPDIR`​：Go工具的临时文件目录。
22. ​`GOTOOLDIR`​：Go工具的安装目录。
23. ​`GOVCS`​：控制版本控制系统的使用。
24. ​`GOVERSION`​：Go的版本。
25. ​`CXX`​：C++编译器。
26. ​`CGO_ENABLED`​：是否启用cgo。
27. ​`GOMOD`​：当前正在使用的Go mod文件的路径。
28. ​`GOWORK`​：Go 1.18引入的新特性，用于支持工作空间模式。
29. ​`CGO_CFLAGS`​、`CGO_CPPFLAGS`​、`CGO_CXXFLAGS`​、`CGO_FFLAGS`​、`CGO_LDFLAGS`​：这些是传递给cgo的编译器标志。
30. ​`PKG_CONFIG`​：pkg-config的路径，pkg-config是一个帮助编译器找到库的工具。
31. ​`GOGCCFLAGS`​：传递给gcc的标志。

​`GOPATH为什么只有bin和pkg?`​

​`我安装的第三方库的源码都存在了pkg目录下`​

从Go 1.11开始，Go引入了模块（Modules）的概念，这是Go包的新的依赖管理系统。在使用模块模式时，Go会将下载的依赖包的源码存放在`GOPATH/pkg/mod`​目录下，而不是`GOPATH/src`​目录下。

你看到的第三方库的源码存在于`pkg`​目录下，可能是因为你的Go环境已经启用了模块模式。你可以通过查看`GO111MODULE`​环境变量的值来确认这一点。如果`GO111MODULE`​的值为`on`​，那么无论你的代码在何处，Go都会使用模块模式。

如果你想要在`GOPATH/src`​目录下看到第三方库的源码，你可以关闭模块模式，将`GO111MODULE`​环境变量的值设置为`off`​。但是请注意，从Go 1.16开始，Go默认启用模块模式，而且在未来的版本中，Go可能会完全移除对`GOPATH`​模式的支持。因此，建议你尽可能地适应使用模块模式。

# go网络编程中conn接口

在Go语言的网络编程中，`net.Conn`​ 是一个核心的接口，它代表了网络连接的端点，可以进行读写操作。以下是 `net.Conn`​ 接口的定义：

```go
type Conn interface {
    // Read方法从连接中读取数据
    Read(b []byte) (n int, err error)

    // Write方法向连接中写入数据
    Write(b []byte) (n int, err error)

    // Close方法关闭连接
    Close() error

    // LocalAddr返回本地网络地址
    LocalAddr() Addr

    // RemoteAddr返回远程网络地址
    RemoteAddr() Addr

    // SetDeadline设置读和写操作的绝对超时时间
    SetDeadline(t time.Time) error

    // SetReadDeadline设置读操作的绝对超时时间
    SetReadDeadline(t time.Time) error

    // SetWriteDeadline设置写操作的绝对超时时间
    SetWriteDeadline(t time.Time) error
}
```

* ​`Read(b []byte) (n int, err error)`​：读取数据到 `b`​ 中，返回读取的字节数和可能的错误。
* ​`Write(b []byte) (n int, err error)`​：将 `b`​ 中的数据写入连接，返回写入的字节数和可能的错误。
* ​`Close() error`​：关闭连接，后续的读写操作都会返回错误。
* ​`LocalAddr() Addr`​：返回本地地址，即连接的一端。
* ​`RemoteAddr() Addr`​：返回远程地址，即连接的另一端。
* ​`SetDeadline(t time.Time) error`​：设置读写操作的超时时间。如果超过这个时间，读写操作就会返回超时错误。
* ​`SetReadDeadline(t time.Time) error`​：设置读操作的超时时间。如果超过这个时间，读操作就会返回超时错误。
* ​`SetWriteDeadline(t time.Time) error`​：设置写操作的超时时间。如果超过这个时间，写操作就会返回超时错误。

这个接口被广泛用于各种网络协议的实现，例如TCP、UDP等。

# 老版本的golang安装使用protobuf(未验证)

gRpc 发布了1.0版，想来试试看，发现新电脑没有装protobuf，之前装的完了记过程，又重新网上搜了一下做个记录,我的系统是ubuntu15.10

获取 Protobuf 编译器 protoc  
我是从github上直接下载的源码编译的，下载地址https://github.com/google/protobuf/releases/tag/v3.0.2，下载后按照文档上的说明操作：

```go
1.检查安装需要用到的编译工具
$ sudo apt-get install autoconf automake libtool curl make g++ unzip 
2.生成需要的配置脚本
$ ./autogen.sh
3.编译安装
$ ./configure
$ make
$ make check
$ sudo make install
$ sudo ldconfig # refresh shared library cache.
```

#### **获取 goprotobuf 提供的 Protobuf 编译器插件 protoc-gen-go**

编译好的插件要放置于 GOPATH/bin 应该被加入 PATH 环境变量，以便 protoc 能够找到 protoc-gen-go

```go
go get github.com/golang/protobuf/protoc-gen-go
进入下载好的目录
go build
```

#### **获取 goprotobuf 提供的支持库，包含诸如编码（marshaling）、解码（unmarshaling）等功能**

```go
go get github.com/golang/protobuf/proto
进入下载好的目录
go build
go install
```

#### **测试一下生成pb.go文件**

```go
//随便找一个.proto文件
package example;

enum FOO { X = 17; };

message Test {
required string label = 1;
optional int32 type = 2 [default=77];
repeated int64 reps = 3;
optional group OptionalGroup = 4 {
required string RequiredField = 5;
}
}
```

编译在.proto同级目录执行，这里通过 –go_out 来使用 goprotobuf 提供的 Protobuf 编译器插件 protoc-gen-go。这时候我们会生成一个名为 *.pb.go 的源文件

> protoc –go_out=. *.proto

补充一下，看过grpc中的例子同学可能会发现除了定义的request和respose以外，在helloworld.pb.go的文档中还生成了，相应client和server的api，后来查了一下是插件protoc-gen-go中做了支持，命令如下：

​`protoc --go_out=plugins=grpc,import_path=mypackage:. *.proto`​

# Git回退历史版本

今天git查看历史版本，回退到了指定历史版本，但是！！我本地代码没有commit。git只能恢复到已经commit的代码或者已经push版本中。  
怎么办呢，我发现idea有一个Local History 功能，可以查看你所有的编辑历史记录，使用这个功能可以回退到本地未保存的代码。  
具体操作如下：

1.右键点击项目名

![image](http://127.0.0.1:6806/assets/image-20230812141020-inwvhpd.png)​

2.点击Local History, 点击show history

![image](http://127.0.0.1:6806/assets/image-20230812141027-m9iygnt.png)​

3.左边是所有的历史编辑记录，点击其中一个右边会展示你修改过的代码

![image](http://127.0.0.1:6806/assets/image-20230812141033-cca76pw.png)

4.选择一个历史记录，右键点击，然后点击Revert,就恢复到本地代码了

![image](http://127.0.0.1:6806/assets/image-20230812141039-hrkr8bg.png)​

# Go数组和切片定义和初始化

　　

**1 前言**

切片是动态数组，数组数组是按值赋值，切片是按地址赋值（引用）

**2 代码**

2.1 数组初始化

```go
func basic_array(){
    //var arr2 = [3]int{2,4,6} // 1
 
    //arr2 := [3]int{2,4,6} //2
       
    //var arr2[3]int = [3]int{2,4,6} //3
 
    //var arr2 [3]int  //4
    //arr2=[3]int{1,3,5}
 
    // var arr2 [3]int //5
    // var i int
    // for i=0;i<len(arr2);i++{
    //  arr2[i] = 10+i;
    // }
 
        // var arr2 [3]int //6
        // var arr2 [...]int{1,3,7} //7[...]表示根据元素自适应大小
 
 
 
    for i,e := range arr2{
        fmt.Println("arr2[",i,"]->",e);
    }
}
```

2.2 切片初始化

```go
func basic_slice(){
 
    //var arr2 = []int{2,4,6} // 1
 
    //arr2 := []int{2,4,6} //2
       
    //var arr2[]int = [3]int{2,4,6} //3
 
    //var arr2 []int  //4
    //arr2=[]int{1,3,5}
 
       //var slice1 []type = make([]type, len) //5
　　　　//slice1 := make([]type, len) //6
　　　　//also add parameter capacity
　　　　//make([]T, length, capacity) //7,8
 
    for i,e := range arr2{
        fmt.Println("arr2[",i,"]->",e);
    }
 
}
```

**3 数组与切片区别**

```go
var arr2[3]int = [3]int{2,4,6}
    var arr1 = arr2
    arr1[1]=11
 
    for i,e := range arr1{
        fmt.Println("arr1[",i,"]->",e);
    }
 
    for i,e := range arr2{
        fmt.Println("arr2[",i,"]->",e);
    }
 
//result
// arr1[ 0 ]-> 2
// arr1[ 1 ]-> 11
// arr1[ 2 ]-> 6
// arr2[ 0 ]-> 2
// arr2[ 1 ]-> 4
// arr2[ 2 ]-> 6
 
 
 
//slice
var arr2[]int = []int{2,4,6}
    var arr1 = arr2
    arr1[1]=11
 
    for i,e := range arr1{
        fmt.Println("arr1[",i,"]->",e);
    }
 
    for i,e := range arr2{
        fmt.Println("arr2[",i,"]->",e);
    }
 
//result
// arr1[ 0 ]-> 2
// arr1[ 1 ]-> 11
// arr1[ 2 ]-> 6
// arr2[ 0 ]-> 2
// arr2[ 1 ]-> 11
// arr2[ 2 ]-> 6
```

# Go内置函数与为什么切片不用初始化就可以使用

Go语言中的内建函数有很多，它们的实现通常是在Go语言的运行时系统中，而不是在Go标准库的源代码中。以下是一些常用的内建函数：

1. ​`append`​：用于向切片添加元素。
2. ​`cap`​：返回切片或数组的容量。
3. ​`close`​：用于关闭通道。
4. ​`complex`​：用于创建一个复数。
5. ​`copy`​：用于复制切片。
6. ​`delete`​​：用于从映射中删除一个键值对。
7. ​`imag`​：返回复数的虚部。
8. ​`len`​：返回切片、数组或字符串的长度。
9. ​`make`​：用于创建切片、映射或通道。
10. ​`new`​：用于创建一个指针。
11. ​`panic`​：用于触发一个运行时错误。
12. ​`print`​和`println`​：用于打印值。
13. ​`real`​：返回复数的实部。
14. ​`recover`​：用于从`panic`​中恢复。

这些函数都是由编译器特殊处理的，它们的实现通常是用汇编语言或C语言编写的，而且可能会针对特定的硬件架构进行优化。因此，这些函数的源代码对大多数Go程序员来说可能难以理解。

---

	我点开append源码后只显示了：func append(slice []Type, elems ...Type) []Type 没有具体源码执行过程是为什么？

​`append`​函数在Go语言中是一个内建函数，它的实现是在Go语言的运行时系统（runtime）中，而不是在Go标准库的源代码中。这就是为什么你在Go标准库的源代码中看不到`append`​函数的具体实现。

内建函数，如`append`​、`make`​、`len`​等，是由编译器特殊处理的。它们的实现通常是用汇编语言或C语言编写的，而且可能会针对特定的硬件架构进行优化。因此，这些函数的源代码对大多数Go程序员来说可能难以理解。

如果你对`append`​函数的实现感兴趣，你可以在Go语言的运行时系统的源代码中找到。但是请注意，这需要对底层的系统编程和计算机硬件有深入的理解。

总的来说，对于大多数Go程序员来说，了解`append`​函数的行为和用法就足够了，无需深入到它的具体实现。

---

	append函数是怎么判断slice是否为nil切片的？

​`append`​函数在内部会检查传入的切片是否为`nil`​。如果是，它会创建一个新的底层数组并返回一个新的切片，这个新的切片引用了新创建的底层数组。

在Go语言的源代码中，`append`​函数的实现大致如下（这是简化的版本，实际的实现更复杂）：

```go
func append(slice []Type, elems ...Type) []Type {
    if slice == nil {
        // 如果切片为nil，创建一个新的底层数组
        slice = make([]Type, 0)
    }
    // 将元素添加到切片的末尾
    // 如果底层数组的容量不足，会自动分配一个更大的底层数组
    return append(slice, elems...)
}
```

这个简化的版本中，你可以看到`append`​函数首先检查切片是否为`nil`​。如果是，它会使用`make`​函数创建一个新的底层数组。然后，它将元素添加到切片的末尾，并返回新的切片。

这就是为什么你可以在不进行初始化的情况下使用切片。当你使用`append`​函数时，如果切片为`nil`​，`append`​函数会自动为你创建一个新的底层数组。

---

在Go语言中，切片（slice）、映射（map）和通道（channel）都是引用类型，它们的零值都是`nil`​。然而，它们的行为在未初始化时是不同的。

对于切片，即使它的零值是`nil`​，你仍然可以使用`append`​函数向其添加元素。这是因为`append`​函数会检查切片是否为`nil`​，如果是，它会分配一个新的底层数组。因此，你可以在不进行初始化的情况下使用切片。

```go
var s []string
s = append(s, "a") // append会检查s是否为nil，如果是，它会分配一个新的底层数组
fmt.Println(s) // 输出 ["a"]
```

然而，对于映射和通道，你必须先进行初始化（即分配内存）才能使用它们。否则，你将会得到一个运行时错误。

对于映射，你必须使用`make`​函数进行初始化：

```go
var m map[string]string
m = make(map[string]string) // 使用make函数初始化映射
m["a"] = "b"
fmt.Println(m) // 输出 map[a:b]
```

对于通道，你也必须使用`make`​函数进行初始化：

```go
var c chan string
c = make(chan string) // 使用make函数初始化通道
go func() {
    fmt.Println(<-c)
}()
c <- "a"
```

总的来说，尽管切片、映射和通道都是引用类型，但是只有切片可以在未初始化的情况下使用。这是因为`append`​函数会自动为`nil`​切片分配新的底层数组。然而，对于映射和通道，你必须先进行初始化才能使用它们。

# go线程与通道

go中的线程也叫goroutine，通道也叫channel

```go
import(
         "fmt"
)

func main(){
    go fmt.Println("1")
    fmt.Println("2")  
}
```

在golang里面，使用go这个关键字，后面再跟上一个函数就可以创建一个线程。后面的这个函数可以是已经写好的函数，也可以是一个匿名函数

```go
func main(){
	var i = 3
	go func (a int){
		fmt.Println( a )
		fmt.Println("1")
	}(i)
	fmt.Println("2")
}
```

上面的代码就创建了一个匿名函数，并且还传入了一个参数i，下面括号里的i是实参，a是形参。那么上面的代码能按照我们预想的打印1、2、3吗？告诉你们吧，不能，程序只能打印出2。下面我把正确的代码贴出来吧

```go
func main(){
	var i = 3
	go func (a int){
		fmt.Println( a )
		fmt.Println("1")
	}(i)

	//time.Sleep(3 * time.Second)
	fmt.Println("2")
	time.Sleep(3 * time.Second) //试一试在上面两行
}
```

```
我只是在最后加了一行让主线程休眠一秒的代码，程序就会依
次打印出2、3、1。
```

那为什么会这样呢？因为程序会优先执行主线程，主线程执行完成后，程序会立即退出，没有多余的时间去执行子线程。如果在程序的最后让主线程休眠1秒钟，那程序就会有足够的时间去执行子线程。

channel:

通道又叫channel，顾名思义，channel的作用就是在多线程之间传递数据的。

创建无缓冲channel

chreadandwrite :=make(chan int)

```go
chonlyread := make(<-chan int)  //创建只读channel
```

```go
chonlywrite := make(chan<- int) //创建只写channel
```

例子：

```go
ch:= make chan(a int)
ch <- 1
go func(){
	<- ch
	fmt.Println("1")
}()
	fmt.Println("2")

```

这段代码执行时会出现一个错误：fatal error: all goroutines are asleep - deadlock!

这个错误的意思是说线程陷入了死锁，程序无法继续往下执行。那么造成这种错误的原因是什么呢？我们创建了一个无缓冲的channel，然后给这个channel赋值了，程序就是在赋值完成后陷入了死锁。因为我们的channel是无缓冲的，即同步的，赋值完成后来不及读取channel，程序就已经阻塞了。这里介绍一个非常重要的概念：channel的机制是先进先出，如果你给channel赋值了，那么必须要读取它的值，不然就会造成阻塞，当然这个只对无缓冲的channel有效。对于有缓冲的channel，发送方会一直阻塞直到数据被拷贝到缓冲区；如果缓冲区已满，则发送方只能在接收方取走数据后才能从阻塞状态恢复。

对于上面的例子有两种解决方案：

1、给channel增加缓冲区，然后在程序的最后让主线程休眠一秒，代码如下：

```go
ch := make (chan int, 1)
ch <- 1
go func(){
	v := <- ch
	fmt.Println(v)
}()
time.Sleep(1 * time.Second)
fmt.Println("2")
```

这样的话程序就会依次打印出1、2

2、利用无缓冲管道特性，在主线程中卡住：

```go
ch := make (chan int)
go func(){
	v := <- ch
	fmt.Println(v)
}()

ch <- 1
fmt.Println("2")
```

这里就不用让主线程休眠了，因为channel在主线程中被赋值后，主线程就会阻塞，直到channel的值在子线程中被取出。

生产者-消费者：

无缓冲版本：

```go
func produce(p chan<- int){
	for i=0; i<=9; i++ {
		p<- i
		fmt.Println("send:",i)
	}
}

func consumer(q <-chan int ){
	for i=0; i<=9; i++ {
		o:= <- q
		fmt.Println("recevice:", o)
	}
}

func main(){
	ch:= make (chan int)
	go produce(ch)
	go sonsumer(ch)
	time.Sleep(1 *time.Second)

}
```

​![image](http://127.0.0.1:6806/assets/image-20230814135343-fraky4g.png)​

有缓冲版本：

```go
func produce(p chan<- int){
	for i=0; i<=9; i++ {
		p<- i
		fmt.Println("send:",i)
	}
}

func consumer(q <-chan int ){
	for i=0; i<=9; i++ {
		o:= <- q
		fmt.Println("recevice:", o)
	}
}

func main(){
	ch:= make (chan int, 10)
	go produce(ch)
	go sonsumer(ch)
	time.Sleep(1 *time.Second)

}
```

​![image](http://127.0.0.1:6806/assets/image-20230814135443-rqjj0u5.png)​

# 字符串截取

Go语言没有像Java一样的substring()方法，但是可以通过如下方式实现字符串截取

```go
func Test_GoSubString(t *testing.T) {
	str := "sssssddddd"
	rs := []rune(str)
	// rs[开始索引:结束索引]
	fmt.Println(string(rs[3:6]))
	str = "你好, Go语言"
	rs = []rune(str)
	fmt.Println(string(rs[1:4]))
}
```

通过将string转为rune数组，获取数组中指定索引区间的元素，就可以实现字符串截取功能  
结果：

ssd  
好,  

# 清空切片的操作

在Go语言中，清空切片（slice）可以通过以下两种方式实现：

1. **重新赋值**：将切片重新赋值为一个空的切片。这是最简单的方法，但请注意，这并不会立即释放原切片的内存。

```go
slice = []int{}
// fmt.Println(sliceIntA==nil) //false
```

2. **使用内置函数**​**`make`**​：使用`make`​函数创建一个新的空切片，并将其赋值给原切片。这也不会立即释放原切片的内存。

```go
slice = make([]int, 0)
```

如果你想立即释放原切片的内存，你可以使用`nil`​：

```go
slice = nil  
// fmt.Println(sliceIntB==nil) //true
```

这将把切片的引用设置为`nil`​，原切片的内存将被垃圾回收器回收。

请注意，以上方法都不会影响其他引用同一底层数组的切片。如果你需要清空所有引用同一底层数组的切片，你需要清空每一个切片。

——使用&quot; slice= nil &quot;处理后有几个关键的问题：

* **要检查切片是否为空，请始终使用**​**`len(s) == 0`**​**来判断**，而不应该使用`s == nil`​来判断。

  * 正如上面的sliceIntA一样，虽然是空切片，但是却不是零值。
  * **一个nil值的切片并没有底层数组，但是一个nil值的切片的长度和容量都是0。但是我们却不能说一个长度和容量都是0的切片一定是nil；**
  * 通过`nil`​清空切片后，切片就没有指向的底层数组，如果没有其他引用这个底层数组，没猜错的话，恐怕只能依靠`GC`​回收了。

### 从地址角度理解

```go
func main() {
	array := [...]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	sliceIntA := array[0:6]
	fmt.Printf("1.array %p\n", &array)
	fmt.Printf("2.sliceIntA %p\n", sliceIntA)
	fmt.Println(sliceIntA)
	sliceIntA = []int{}
	fmt.Printf("1.array  %p\n", &array)
	fmt.Printf("2.sliceIntA %p\n", sliceIntA)
	sliceIntA = nil
	fmt.Printf("2.sliceIntA %p\n", sliceIntA)
	fmt.Printf("1.array  %p\n", &array)
	sliceIntA = append(sliceIntA, 1)
	fmt.Printf("2.sliceIntA %p\n", sliceIntA)
}
```

```go
1.array 0xc00000c1e0
2.sliceIntA 0xc00000c1e0
[1 2 3 4 5 6]
1.array  0xc00000c1e0
 
# 清空方法一
2.sliceIntA 0x108de00
 
# 清空方法二
2.sliceIntA 0x0
1.array  0xc00000c1e0
 
# append
2.sliceIntA 0xc0000120d0
```

可以看出：

* 使用方法一清空后，切片指向的底层数组更改了，原有的底层数组不变，因此此时再操作切片，不会影响原有底层数组；
* 使用nil清空后，切片就没有底层数组了，append后，就指向了新的底层数组，原有的底层数组不变；

**由此可以得到结论**：以上清空方式，均会导致底层更换数组。

‍

产生的问题：

* **在需要清空继续append操作的情况下，均会导致底层更换数组，开辟新的空间，原有底层数组恐怕依靠GC回收了；**
* 切片清空后，除了长度归0，容量也归0了，这其实并不利于我们后续`append`​，**因为当长度即将大于容量时，就会按照扩容策略进行扩容。**

### 更优雅的方式：

在Go中，如果你想清空一个切片但又想保留其底层数组的内存，你可以使用切片的切片操作来实现。这种方式可以避免内存的重新分配，特别适合于大切片的清空。

```go
slice = slice[:0]
```

这行代码将切片的长度设置为0，但保留其容量。这意味着切片看起来像是被清空了，但其底层数组的内存实际上是被保留了下来的。如果你之后又向切片中添加元素，只要添加的元素数量不超过切片的容量，就不会发生内存的重新分配。

这种方式的优点是：效率高（特别是对于大切片），但需要注意的是，由于底层数组的内存被保留了下来，如果你不再需要这些内存，可能会导致内存浪费。如果你确定不再需要这些内存，应该使用`slice = nil`​或`slice = make([]T, 0)`​来释放内存。

### 优雅方式背后的细节

```go
func main() {
	array := [...]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	sliceIntA := array[0:6]
	fmt.Printf("array %p\n", &array)
	fmt.Printf("sliceIntA %p\n", sliceIntA)
	fmt.Println(array)
	fmt.Println(sliceIntA)
	fmt.Printf("len of sliceIntA:%d,cap of sliceIntA:%d\n", len(sliceIntA), cap(sliceIntA))
	sliceIntA = sliceIntA[0:0]
	fmt.Printf("array %p\n", &array)
	fmt.Printf("sliceIntA %p\n", sliceIntA)
	fmt.Println(sliceIntA)
	fmt.Println(array)
	fmt.Printf("len of sliceIntA:%d,cap of sliceIntA:%d\n", len(sliceIntA), cap(sliceIntA))
}
```

```go
array 0xc00000c1e0
sliceIntA 0xc00000c1e0
[1 2 3 4 5 6 7 8 9 10]
[1 2 3 4 5 6]
len of sliceIntA:6,cap of sliceIntA:10
#清空后
array 0xc00000c1e0
sliceIntA 0xc00000c1e0
[]
[1 2 3 4 5 6 7 8 9 10]
len of sliceIntA:0,cap of sliceIntA:10
```

通过输出，我们很容易得到如下结论：

* 通过**切片表达式清空**切片，**仅长度归0，而容量维持不变**

  * 解决了可能扩容的问题
* 清空后，切片指向的底层数组也不变

  * 解决了更换底层数组，开辟新空间，以及可能的垃圾回收问题

**注意**：切片指向的底层数组不变，也就导致了无论是通过切片操作还是数组操作，都会影响彼此。

```go
sliceIntA = append(sliceIntA, 999) //切片清空后append值
sliceIntA = append(sliceIntA, 888) //切片清空后append值
fmt.Printf("array %p\n", &array)
fmt.Printf("sliceIntA %p\n", sliceIntA)
fmt.Println(sliceIntA) //[999 888]
fmt.Println(array) //[999 888 3 4 5 6 7 8 9 10]
fmt.Printf("len of sliceIntA:%d,cap of sliceIntA:%d\n", len(sliceIntA), cap(sliceIntA))
array[0]=777  //通过数组索引修改值
fmt.Printf("array %p\n", &array)
fmt.Printf("sliceIntA %p\n", sliceIntA)
fmt.Println(sliceIntA)  // [777 888]
fmt.Println(array) //[777 888 3 4 5 6 7 8 9 10]
fmt.Printf("len of sliceIntA:%d,cap of sliceIntA:%d\n", len(sliceIntA), cap(sliceIntA))
```

```go
array 0xc00000c1e0
sliceIntA 0xc00000c1e0
[999 888]
[999 888 3 4 5 6 7 8 9 10]
len of sliceIntA:2,cap of sliceIntA:10
array 0xc00000c1e0
sliceIntA 0xc00000c1e0
[777 888]
[777 888 3 4 5 6 7 8 9 10]
len of sliceIntA:2,cap of sliceIntA:10
```

看，此时修改切片会影响数组，修改数组会影响切片，直到**切片长度即将超越容量，底层数组更换，它俩才会脱钩**

### 结论

算下来就有3种清空切片的方法，但是他们的本质各不相同，对性能没啥极致的要求，都可使用，如果还有其他特别的需求，一定要留心底层数组是否更换，请小心使用。当然，如果仅仅是清空切片，后续继续 `append`​ 操作，博主更推荐切片表达式的方法`[0:0]`​ 。

# map清空操作

在Go语言中，清空一个map的方法是通过重新为其分配内存。这是因为Go没有提供直接清空map的内置函数。以下是一个示例：

```go
package main

import "fmt"

func main() {
    m := map[string]int{"one": 1, "two": 2, "three": 3}
    fmt.Println(m) // 输出: map[one:1 three:3 two:2]

    m = make(map[string]int)
    fmt.Println(m) // 输出: map[]
}
```

在这个例子中，我们首先创建了一个map `m`​，然后通过`m = make(map[string]int)`​重新为其分配内存，这样原来的map就被清空了。

除了重新分配内存，另一种清空map的方法是遍历map并删除每个元素。这种方法的效率较低，但在某些情况下可能会有用，例如当你需要保持map的内存分配不变时。以下是一个示例：

```go
package main

import "fmt"

func main() {
    m := map[string]int{"one": 1, "two": 2, "three": 3}
    fmt.Println(m) // 输出: map[one:1 three:3 two:2]

    for k := range m {
        delete(m, k)
    }
    fmt.Println(m) // 输出: map[]
}
```

在这个例子中，我们使用了`for`​循环和`delete`​函数来删除map中的每个元素。这样，map就被清空了，但其内存分配保持不变。

在Go语言中，清空一个map最常用且最直接的方法就是重新为其分配内存，即`m = make(map[KeyType]ValueType)`​。这种方法既简单又高效，因为它直接丢弃了旧的map，并创建了一个新的空map。

如果你希望保持map的内存分配不变，那么你可以遍历map并删除每个元素。但这种方法的效率较低，因为它需要遍历整个map。

# gin框架返回json数据两种方法

我们在开发是传送给前端的数据往往是以json格式发送的，但具体的方法是有一下两种 （map和结构体）

```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

func main() {
    //设置返回路由引擎
    r := gin.Default()

    r.GET("/json", func(c *gin.Context) {
        //方法一：map
        // data := map[string]interface{}{
        //     "name":    "小魔仙",
        //     "message": "hello golang",
        //     "age":     13,
        //     "sex":     "男",
        // }
        c.JSON(http.StatusOK, gin.H{
            "name":    "小魔仙",
            "message": "hello golang",
            "age":     13,
            "sex":     "女",
            "b1":      " 今天是2021年08-30",
        })
    })
    //结构体返回，灵活使用tag来对结构体字段做定制化操作
    type msg struct {
        Name    string
        Message string
        Age     int
    }
    r.GET("/another_json", func(c *gin.Context) {
        data := msg{
            Name:    "乔四美",
            Message: "七七",
            Age:     77,
        }
        c.JSON(http.StatusOK, data) //json的序列化，通过反射 ，所以结构体内的字段首字母要大写
    })

    r.Run(":9090")
}
```

# 判断结构体是否为空

在Go语言中，判断一个结构体是否为空并不像判断一个字符串或切片是否为空那么直接。这是因为结构体的零值（即所有字段都是其类型的零值）并不等同于nil。以下是一些常见的判断结构体是否为空的方法：

1. **比较结构体与其类型的零值**：你可以创建一个该类型的零值，并将你的结构体与之比较。如果它们相等，那么你的结构体就是空的。

```go
type MyStruct struct {
    Field1 int
    Field2 string
}

func main() {
    var s MyStruct
    if s == (MyStruct{}) {
        fmt.Println("s is empty")
    } else {
        fmt.Println("s is not empty")
    }
}
```

2. **检查结构体的所有字段**：你可以检查结构体的所有字段是否都是其类型的零值。如果是，那么你的结构体就是空的。这种方法在结构体有很多字段或字段类型复杂时可能会很繁琐。

```go
type MyStruct struct {
    Field1 int
    Field2 string
}

func main() {
    var s MyStruct
    if s.Field1 == 0 && s.Field2 == "" {
        fmt.Println("s is empty")
    } else {
        fmt.Println("s is not empty")
    }
}
```

3. **使用指向结构体的指针**：如果你的结构体可能为空，并且你希望能够区分空结构体和非空结构体，那么你可以使用指向结构体的指针。这样，你可以使用nil来表示空结构体。

```go
type MyStruct struct {
    Field1 int
    Field2 string
}

func main() {
    var s *MyStruct
    if s == nil {
        fmt.Println("s is empty")
    } else {
        fmt.Println("s is not empty")
    }
}
```

在这个例子中，`s`​是一个指向`MyStruct`​的指针，其零值是nil。因此，你可以使用`s == nil`​来检查`s`​是否为空。

4. **反射 reflect.DeepEqual**

    ```go
    package main

    import (
    	"fmt"
    	"reflect"
    )

    type A struct {
    	name string
    	age  int
    }

    func (a A) IsEmpty() bool {
    	return reflect.DeepEqual(a, A{})
    }

    func main() {
    	var a A

    	if a == (A{}) { // 括号不能去
    		fmt.Println("a == A{} empty")
    	}

    	if a.IsEmpty() {
    		fmt.Println("reflect deep is empty")
    	}
    }
    ```

​`reflect.DeepEqual`​函数可以用来比较两个值是否深度相等，包括结构体。这意味着你可以使用`reflect.DeepEqual`​来判断一个结构体是否为空，即是否等于其类型的零值。以下是一个示例：

```go
package main

import (
	"fmt"
	"reflect"
)

type MyStruct struct {
	Field1 int
	Field2 string
}

func main() {
	var s MyStruct
	if reflect.DeepEqual(s, MyStruct{}) {
		fmt.Println("s is empty")
	} else {
		fmt.Println("s is not empty")
	}
}
```

在这个例子中，我们使用`reflect.DeepEqual(s, MyStruct{})`​来检查结构体`s`​是否为空。如果`s`​等于`MyStruct{}`​（即`MyStruct`​的零值），那么`reflect.DeepEqual`​将返回`true`​，否则它将返回`false`​。

​`reflect.DeepEqual`​函数是一个非常强大的工具，它可以比较任何两个值，包括复杂的结构体和嵌套的切片或映射。然而，它也有一些限制，例如它不能比较函数或未导出的字段。因此，你应该根据具体的需求来选择最适合的方法。

# 数组和切片的区别

在Go语言中，数组和切片是两种不同的序列类型，它们有以下主要的区别：

1. 长度：

    * 数组：在声明时需要指定长度，并且长度是数组类型的一部分。一旦声明，数组的长度就不能改变。
    * 切片：长度是动态的，可以在运行时改变。切片的长度可以通过内置的`len`​函数获取，可以通过`append`​函数增加。
2. 容量：

    * 数组：容量就是数组的长度，不能改变。
    * 切片：容量是底层数组的长度，可以通过内置的`cap`​函数获取。切片的容量通常大于或等于切片的长度，当通过`append`​函数增加切片的长度时，如果超过当前的容量，Go会自动分配一个新的底层数组，并复制原来的元素到新数组。
3. 值传递：

    * 数组：当数组作为函数参数时，会创建一个数组的副本，修改副本不会影响原数组。
    * 切片：当切片作为函数参数时，虽然也是值传递，但由于切片的值包含一个指向底层数组的指针，所以修改切片的元素会影响原切片。
4. 内存使用：

    * 数组：由于数组的长度是固定的，所以数组通常使用较少的内存。
    * 切片：由于切片的长度和容量是动态的，所以切片可能会使用更多的内存。特别是当切片的长度远小于其容量时，可能会浪费大量的内存。
5. 初始化方式：

    * Go语言中的数组和切片的初始化方法是不完全相同的。以下是数组和切片的初始化方法：

      1）数组在声明时需要指定固定的长度。数组的初始化可以通过以下方式：

      ```go
      // 声明并初始化数组
      var arr1 [5]int = [5]int{1, 2, 3, 4, 5}
      // 简化声明
      arr2 := [5]int{1, 2, 3, 4, 5}
      // 让Go语言自动推断数组长度
      arr3 := [...]int{1, 2, 3, 4, 5}
      ```

      2） 切片在声明时不需要指定长度，因为切片是动态的。切片的初始化可以通过以下方式：

      ```go
      // 声明并初始化切片
      var slice1 []int = []int{1, 2, 3, 4, 5}
      // 简化声明
      slice2 := []int{1, 2, 3, 4, 5}
      // 使用make函数创建切片
      slice3 := make([]int, 5) // 创建一个长度为5的切片，元素全部初始化为0
      ```

      需要注意的是，切片是对数组的抽象，它实际上是对底层数组的引用。因此，切片的操作可能会影响底层数组的内容。
    * **数组的初始化**：
    * **切片的初始化**：
6. 使用场景：

    * 数组：适合于长度固定，且不需要改变的场景。
    * 切片：适合于长度可变，需要动态添加或删除元素的场景。

总的来说，数组和切片都是Go语言中重要的数据结构，它们各有优点和用途，需要根据具体的需求来选择使用哪一种。

---

在Go语言中，所有的函数参数都是值传递，包括数组和切片。这意味着当你将一个变量传递给函数时，函数会接收到这个变量的一个副本，而不是原始变量。如果你在函数内部修改这个副本，原始变量不会受到影响。

然而，数组和切片在值传递时的行为是不同的：

* 数组：当你将一个数组传递给函数时，函数会接收到数组的一个完整副本。这个副本包含了数组的所有元素。如果你在函数内部修改这个副本，原始数组不会受到影响。
* 切片：当你将一个切片传递给函数时，函数会接收到切片的一个副本。但是，这个副本并不包含切片的元素，而是包含了一个指向原始数组的指针、切片的长度和容量。这意味着如果你在函数内部修改切片的元素，原始切片的元素也会被修改。但是，如果你在函数内部改变切片的长度或容量（例如，通过append或copy函数），原始切片不会受到影响。

  ——上述的copy函数会改变，append不会变

  以下是一个示例来说明这个概念：

  ```go
  package main

  import "fmt"

  func modifySliceAppend(slc []int) {
      slc = append(slc, 4)
  }

  func modifySliceCopy(slc []int) {
      slc2 := []int{4, 5, 6}
      copy(slc, slc2)
  }

  func main() {
      slc := []int{1, 2, 3}

      modifySliceAppend(slc)
      fmt.Println(slc) // 输出 [1 2 3], 切片的长度和容量没有被修改

      modifySliceCopy(slc)
      fmt.Println(slc) // 输出 [4 5 6], 切片的元素被修改，但长度和容量没有被修改
  }
  ```

  在这个例子中，`modifySliceAppend`​函数试图通过`append`​函数来改变切片的长度和容量，但是由于切片是值传递，所以原始切片的长度和容量并没有被修改。

  而`modifySliceCopy`​函数通过`copy`​函数修改了切片的元素，由于切片的值包含了一个指向原始数组的指针，所以原始切片的元素也被修改了。但是，`copy`​函数并没有改变切片的长度和容量，所以原始切片的长度和容量没有被修改。

  这就是为什么说在Go中，如果你在函数内部改变切片的长度或容量（例如，通过append或copy函数），原始切片不会受到影响。

---

* 数组：当数组作为函数参数时，会创建一个数组的副本，修改副本不会影响原数组。
* 切片：当切片作为函数参数时，虽然也是值传递，但由于切片的值包含一个指向底层数组的指针，所以修改切片的元素会影响原切片。

以下是一些示例来说明这个概念：

```go
package main

import "fmt"

func modifyArray(arr [3]int) {
    arr[0] = 100
}

func modifySlice(slc []int) {
    slc[0] = 100
}

func main() {
    arr := [3]int{1, 2, 3}
    slc := []int{1, 2, 3}

    modifyArray(arr)
    fmt.Println(arr) // 输出 [1 2 3], 数组没有被修改

    modifySlice(slc)
    fmt.Println(slc) // 输出 [100 2 3], 切片被修改
}
```

在这个例子中，`modifyArray`​函数试图修改传入的数组的第一个元素，但是由于数组是值传递，所以原始数组并没有被修改。

而`modifySlice`​函数修改了传入的切片的第一个元素，由于切片的值包含了一个指向原始数组的指针，所以原始切片的元素也被修改了。

---

这就是为什么说在Go中，数组是值传递，切片也是值传递，但是由于切片的值包含了一个指向原始数组的指针，所以在函数内部修改切片的元素会影响原始切片。

所以，更准确的说法是：在Go中，数组是值传递，切片也是值传递，但是由于切片的值包含了一个指向原始数组的指针，所以在函数内部修改切片的元素会影响原始切片。

# Unicode和UTF-8区别

字符集为每个字符分配一个唯一的 ID，我们使用到的所有字符在 Unicode 字符集中都有一个唯一的 ID，例如上面例子中的 a 在 Unicode 与 ASCII 中的编码都是 97。汉字“你”在 Unicode 中的编码为 20320，在不同国家的字符集中，字符所对应的 ID 也会不同。而无论任何情况下，Unicode 中的字符的 ID 都是不会变化的。

Unicode和UTF-8都是用于字符编码的标准，但它们在使用和实现上有一些区别：

1. **Unicode**：Unicode是一个字符集，它为世界上的每种语言的每个字符定义了一个唯一的数字。Unicode可以容纳超过100万个字符。每个Unicode字符都由一个码点（Code Point）表示，码点是一个4字节的数字，用于唯一标识一个字符。
2. **UTF-8**：UTF-8是一种字符编码方案，它是Unicode的一种实现方式。UTF-8使用1到4个字节来表示一个字符，对于ASCII字符，UTF-8使用1个字节，对于其他语言的字符，UTF-8可能使用2到4个字节。UTF-8的优点是它对ASCII字符的兼容性很好，而且它是变长的，可以节省存储空间。

总的来说，Unicode是一个字符集，它定义了每个字符的唯一数字，而UTF-8是一种字符编码方案，它定义了如何将Unicode字符转换为二进制数据。

# go中的字符类型

Go语言中，字符类型主要有两种：byte和rune。

1. **byte**：byte是uint8的别名，它用于表示ASCII字符。byte类型的变量可以存储一个ASCII字符。

```go
var ch1 byte = 'A'
```

2. **rune**：rune是int32的别名，它用于表示一个Unicode字符。rune类型的变量可以存储一个Unicode字符。

```go
var ch2 rune = '你'
```

需要注意的是，Go语言中的字符串是由byte组成的，如果你需要处理Unicode字符，你可能需要使用rune切片。

```go
s := "Hello, 世界"
r := []rune(s)
```

在这个例子中，r是一个rune切片，它可以用来处理包含Unicode字符的字符串。

# go中嵌套结构体赋值

本质上与匿名结构体的使用有关系

```go
type Respone struct {
    i     int
    str     string
    InStru []struct {
        a     int
        strIn string
    }
}
```

一般的方式：

```go
package main

import "fmt"

type Response struct {
    i     int
    str   string
    InStru []struct {
        a     int
        strIn string
    }
}

func main() {
    var res Response
    res.i = 10
    res.str = "Hello"
    res.InStru = append(res.InStru, struct {
        a     int
        strIn string
    }{a: 20, strIn: "World"})

    fmt.Println(res)
}
```

当然，你可以在声明结构体变量的同时进行初始化，这样可以更简洁地为结构体赋值。以下是如何做到这一点的示例：

```go
package main

import "fmt"

type Response struct {
    i     int
    str   string
    InStru []struct {
        a     int
        strIn string
    }
}

func main() {
    res := Response{
        i:   10,
        str: "Hello",
        InStru: []struct {
            a     int
            strIn string
        }{
            {a: 20, strIn: "World"},
        },
    }

    fmt.Println(res)
}
```

在这个例子中，我们在声明Response类型的变量res的同时进行了初始化。我们为res的i和str字段赋值，并为res的InStru字段添加了一个匿名结构体的实例。这个匿名结构体的实例的a和strIn字段分别被赋值为20和"World"。

# 网页计数器

前几日写一个网页的简单计数器问题时发现，计数器居然永远为0，计数器不计数，见鬼了。。。

代码如下：

```go
type Counter struct {
	n int
}

func (ctr Counter) ServeHTTP(c http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(c, "%08x\n", ctr)
	ctr.n++
	fmt.Fprintf(c, "counter = %d\n", ctr.n)
}

func main() {
	http.Handle("/counter", new(Counter))
	log.Fatal("ListenAndServe: ", http.ListenAndServe(":80", nil))
}
```

研究一番，发现我们

```go
func (ctr Counter) ServeHTTP(c http.ResponseWriter, req *http.Request) 
```

应该改为：

```go
func (ctr* Counter) ServeHTTP(c http.ResponseWriter, req *http.Request) 
```

也就是说，对象的实例必须定义为指针的类型，然后才能传递正确的地址，否则ctr参数只是对象的一个副本

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

type Counter struct {
	n int
}

func main() {
	http.Handle("/counter", new(Counter))
	log.Fatal("ListenAndServe: ", http.ListenAndServe(":80", nil))
}


func (ctr *Counter) ServeHTTP(c http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(c, "%08x\n", ctr)
	ctr.n++
	fmt.Fprintf(c, "counter = %d\n", ctr.n)
}

```

​![image](http://127.0.0.1:6806/assets/image-20230815155604-a40l79n.png)​

# 测试函数

​`testing.T`​和`testing.B`​是在测试函数中使用的，它们不能在`main`​函数中使用，也不能在`main`​函数外部使用。它们是由Go的测试框架在运行测试函数时传入的。

​`testing.T`​是在普通的测试函数中使用的，这些测试函数的名字以`Test`​开头。例如：

```go
func TestMyFunction(t *testing.T) {
    // 使用t来报告测试结果
}
```

​`testing.B`​是在基准测试函数中使用的，这些测试函数的名字以`Benchmark`​开头。基准测试用于测量一段代码的性能。例如：

```go
func BenchmarkMyFunction(b *testing.B) {
    for i := 0; i < b.N; i++ {
        // 运行需要测量性能的代码
    }
}
```

这些测试函数应该写在一个名为`*_test.go`​的文件中，这个文件需要和被测试的代码文件在同一个包中。你可以使用`go test`​命令来运行测试。

# 缓存替换算法

以下是对你提到的各种缓存替换算法的简单介绍：

1. LRU（Least Recently Used）：这种算法会淘汰最长时间未被访问的数据。其核心思想是如果一个数据在最近一段时间没有被访问到，那么在将来一段时间内被访问的可能性也很小。
2. LFU（Least Frequently Used）：这种算法会淘汰在一段时间内被访问次数最少的数据。其核心思想是如果一个数据在最近一段时间内被访问次数很少，那么在将来一段时间内被访问的可能性也很小。
3. FIFO（First In First Out）：这种算法会淘汰最早进入缓存的数据。其核心思想是先进先出，这是最简单的一种缓存替换策略。
4. ARC（Adaptive Replacement Cache）：这种算法是LRU和LFU的一种折衷方案，它会根据数据的访问频率和访问时间动态地调整数据的优先级。
5. Random：这种算法会随机淘汰缓存中的数据。其核心思想是每个数据被淘汰的概率是相等的。
6. NearlyLRU：这种算法是LRU的一种变种，它会淘汰近似最近最少使用的数据。与LRU相比，NearlyLRU的实现更简单，但可能会牺牲一些命中率。

以上这些算法各有优缺点，适用于不同的场景。在选择缓存替换算法时，需要根据具体的应用场景和需求来决定。

‍