---
title: 多个协程打印相关问题
date: '2023-10-12 16:08:10'
updated: '2023-10-12 17:06:23'
excerpt: |-
  加深一些并发问题中协程的理解以及channel等部分...
  其实可以联想到业务中，或者现实中的流水线场景，有一组任务要做，这个任务要有甲乙丙三个组件（人）来做，每个（组件）人都要参与实现...
categories:
  - 算法加练
permalink: /post/concorded-programming-multiple-corporate-printing-zpbnes.html
comments: true
toc: true
---

# 多个协程打印相关问题

​`创建三个goroutine，分别输出1 4 7, 2 5 8, 3 6 9, ...... 100， 保证顺序输出1到100`​

```go
/**
 * @Author 560463
 * @Description //TODO $ 并发：创建三个goroutine，分别输出1 4 7, 2 5 8, 3 6 9, ...... 100， 保证顺序输出1到100, 2种方法
 * @Date 2023/10/12 14:36
 **/

// 实现三个协程并发交替打印数字
// 方法一：
package main

import "fmt"

var chanDone chan int
var end int
var countA, countB, countC int

func write(name string, count int, ch, next chan int) {
	for a := range ch {
		//fmt.Println(a)
		count++
		fmt.Printf("协程%s 第%d次打印了%d\n", name, count, a)

		if a < end {
			next <- a + 1
		} else {
			chanDone <- 0
		}
	}
}

func main() {
	ch1, ch2, ch3 := make(chan int, 1), make(chan int, 1), make(chan int, 1)
	countA, countB, countC = 0, 0, 0
	chanDone, end = make(chan int), 100
	go write("A", countA, ch1, ch2)
	go write("B", countB, ch2, ch3)
	go write("C", countC, ch3, ch1)
	ch1 <- 1
	<-chanDone
}

```

‍

```go
//方法二：
package main

import "fmt"

var wg sync.WaitGroup

func r(name string, count int, start int, ch1, ch2 chan int) {
	defer wg.Done()
	for i := start; i <= 100; {
		num, ok := <-ch1
		if !ok {
			close(ch2)
			break
		}
		count++
		fmt.Printf("协程%s 第%d次打印了: %d\n", name, count, num)
		i++
		ch2 <- i
		i = i + 2
		//如果这样写有个坑：这样写第一次A是打印了1 但是给B的值也是1，虽然后面进行了+3 但是本轮打印时错误的
		//ch2 <- i
		//i = i + 3
	}
}

func main() {
	chA, chB, chC := make(chan int, 1), make(chan int, 1), make(chan int, 1)
	countA, countB, countC := 0, 0, 0
	wg.Add(3)
	chA <- 1
	go r("A", countA, 1, chA, chB)
	go r("B", countB, 2, chB, chC)
	go r("C", countC, 3, chC, chA)
	wg.Wait()
}
```

‍

结果：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311221517190.png)​

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311221517938.png)​

‍

---

‍

​`三个协程分别打印100次 cat dog fish`​

```go
package main

import (
	"fmt"
	"sync"
)

// 三个协程交替打印 cat dog fish
var repeatCount = 100

func main() {
	// wg 用来防止主协程提前先退出
	wg := &sync.WaitGroup{}
	wg.Add(3)

	chCat := make(chan struct{}, 1)
	defer close(chCat)
	chDog := make(chan struct{}, 1)
	defer close(chDog)
	chFish := make(chan struct{}, 1)
	defer close(chFish)

	go printAnimal(chCat, chDog, "cat", wg)
	go printAnimal(chDog, chFish, "dog", wg)
	go printAnimal(chFish, chCat, "fish", wg)
	chCat <- struct{}{}
	wg.Wait()
}

// wg 需要传指针
func printAnimal(in, out chan struct{}, s string, wg *sync.WaitGroup) {
	count := 0
	for {
		<-in
		count++
		fmt.Printf("第%d次打印%s\n", count, s)
		out <- struct{}{}

		if count >= repeatCount {
			wg.Done()
			return
		}

	}
}

```

‍

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202310121711529.png)​
