---
title: Go并发编程 | 总纲
date: '2023-12-06 17:50:18'
updated: '2023-12-26 14:09:35'
excerpt: 我的并发编程系列总纲
tags:
  - golang
  - 并发编程
categories:
  - Golang
  - 并发编程
permalink: /post/go-parallel-programming-chief-outline-znxfpt.html
comments: true
toc: true
---

# Go并发编程 | 总纲

> 本专题参考以晁岳攀老师为主，以培养全面、系统的Go并发编程学习体系。
>
> 期间可能也会掺杂一些个人理解或者其他的参考资料进行补充和校正。
>
> 持续更新中······ 预计2024年初更新完

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312071403985.png)​

# 基本介绍

<span style="font-weight: bold;" data-type="strong">基本并发原语</span>：Mutex、RWMutex、Waitgroup、Cond、Pool、Context 等标准库中的并发原语，这些都是传统的并发原语，在其它语言中可能也会有。一通百通

<span style="font-weight: bold;" data-type="strong">原子操作</span>：原子操作是其它并发原语的基础，学会了你就可以自己创造新的并发原语。

<span style="font-weight: bold;" data-type="strong">Channel</span>：Channel 类型是 Go 语言独特的类型，因为比较新，较难以掌握，掌握它的基本用法、处理场景和应用模式，避免踩坑。

<span style="font-weight: bold;" data-type="strong">扩展并发原语</span>：目前来看，Go 开发组不准备在标准库中扩充并发原语了，但是还有一些并发原语应用广泛，比如信号量、SingleFlight、循环栅栏、ErrGroup 等。掌握了它们，就可以在处理一些并发问题时，取得事半功倍的效果。

<span style="font-weight: bold;" data-type="strong">分布式并发原语</span>：分布式并发原语是应对大规模的应用程序中并发问题的并发类型。我主要会介绍使用 etcd 实现的一些分布式并发原语，比如 Leader 选举、分布式互斥锁、分布式读写锁、分布式队列等，在处理分布式场景的并发问题时，特别有用。这一部分结合了分布式的知识，将并发和分布式结合可能会受益匪浅。

# 目标

<u>学习主线层面，主要是四大步骤，包括</u>​<u>基础用法、实现原理、易错场景、知名项目中的 Bug</u><u>。每一个模块，我都会带着你按照这四个步骤来学习，目的就是带你熟知每一种并发原语的实现机制和适用场景</u>

实际上，针对同一种场景，也许存在很多并发原语都适用的情况，但是一定是有最合适的那一个。所以，你必须非常清楚每种并发原语的实现机制和适用场景，千万不要被网上的一些文章误导，万事皆用 Channel

奔着精通的方向：

深入学习下 Go 并发原语的源代码。你会发现很多独到的设计，比如Mutex 为了公平性考量的设计、sync.Map 为提升性能做的设计，以及很多并发原语的异常状况的处理方式。尤其是这些异常状况，常常是并发编程中程序 panic 的原因。

没有做过大型并发项目：可能不理解并发原语的重要性。——>参看知名项目中犯过的错，站在巨人的肩膀上分析总结

怎么算并发编程学的好，精通了？

<span style="font-weight: bold;" data-type="strong">晁岳攀老师给出的建议是：自己创造并发原语库和实现机制以及适用场景。</span> 

‍
