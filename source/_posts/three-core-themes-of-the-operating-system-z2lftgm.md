---
title: 操作系统三大核心主题
date: '2024-01-08 13:47:25'
updated: '2024-01-08 16:34:00'
excerpt: 虚拟化、持久化、并发是操作系统重点关注的三大主题
tags:
  - 操作系统
categories:
  - 计算机底层
  - 操作系统
permalink: /post/three-core-themes-of-the-operating-system-z2lftgm.html
comments: true
toc: true
---



> 今天看博主推文，得知《操作系统导论》一书，感觉有点东西特此记录一下。主要涉及了<span style="font-weight: bold;" data-type="strong">虚拟化、并发、持久化</span>三个重点主题，其中的虚拟化最近比较感兴趣，后续再结合刘丹冰老师的golang内存篇再仔细看看。希望后续可以有自己的理解
>
> 相关文章：
>
> [3.5 万字 + 60 张图 ｜一步一图带你深入理解 Linux 虚拟内存管理](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&amp;mid=2247522087&amp;idx=2&amp;sn=fe8f4cd34d68e0a10658dee88bd337df&amp;chksm=f98dd38dcefa5a9ba43a9d1ac96852532f53278915a6f6b9f5187b8c1c885c1e5848ebabbc86#rd)

作为一个合格的程序员，计算机组成原理、数据结构、算法、网络和操作系统的理论知识是基础中的基础。正所谓，万变不离其宗，只有基础打好了，在新的编程技术层出不穷的时代，才会利于不变之地。

《操作系统导论》是[美]雷姆兹·H·阿帕希杜塞尔、[美]安德莉亚·C·阿帕希杜塞尔夫妇共同编写的一遍关于操作系统基础知识的书。本书深入的讲解了操作系统的三大主题：<span style="font-weight: bold;" data-type="strong">虚拟化、并发和持久化</span>。

<span style="font-weight: bold;" data-type="strong">操作系统本质上也是软件</span>，<span style="font-weight: bold;" data-type="strong">但它介于应用软件和硬件之间</span>。<span style="font-weight: bold;" data-type="strong">其作用就是管理电脑上的所有硬件资源，并把这些硬件资源合理的分配给其上的应用软件</span>。<span style="font-weight: bold;" data-type="strong">同时，它还提供了一组API接口供其上的应用软件来调用，以便让应用软件来使用其下的硬件资源</span>。

操作系统承上启下，向下开发管理分配硬件资源，向上提供API接口供软件或者程序进行硬件资源调用

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202401081635768.png)​

<span style="font-weight: bold;" data-type="strong">简而言之，操作系统的功能就是让应用程序更易于使用以及高效的运行在计算机（或叫电脑）的硬件资源上。</span> 

那么，操作系统是如何只在一组硬件基础之上，让尽量多的应用程序同时高效的运行的呢？那就是<span style="font-weight: bold;" data-type="strong">虚拟化</span>。就是说操作系统会将物理资源（CPU、内存或磁盘）进行转换，转换成更通用、更强大的虚拟形式。

# 虚拟化CPU

虚拟化CPU终究还是想通过分时复用提高利用率

假设机器只有一个CPU。那么，要想让机器“同时”运行多个程序，比如听歌和打游戏，就需要<span style="font-weight: bold;" data-type="strong">在操作系统和硬件的协同下，将单个CPU转换成看似是无数多个CPU，让每个程序都认为自己有一个CPU，这就是所谓的CPU虚拟化</span>。

<span style="font-weight: bold;" data-type="strong">虚拟化CPU的目的就是为了让机器能够同时运行多个程序，提高CPU的利用率</span>。

<span style="font-weight: bold;" data-type="strong">CPU虚拟化的基本原理就是分时复用</span>。CPU一个时刻只能运行一个程序，要想达到同时运行A和B的目的，就需要一个时刻内CPU运行程序A，然后下一个时刻运行程序B，一直这样不停切换，由于CPU的运行速度相对于人来说非常快，就达到了看似同时运行程序的效果。

​![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/l6hSQtEH25iccnGOBic4Dia1H90kPnj46FG9mGN5BfekZ05P41jzzGOIBucqlWy3HicQylONnia8vHLSUFcBOh7neKg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)​

下面通过一个C语言的例子来说明虚拟化CPU后的效果。

```c
#include <stdio.h> 
#include <stdlib.h> 
#include <sys/time.h> 
#include <assert.h> 
#include "common.h"  
int  main(int argc, char *argv[]) 
	{ 
 if (argc != 2) { 
   fprintf(stderr, "usage: cpu <string>\n"); 
   exit(1); 
  } 
  char *str = argv[1]; 
  while (1) { 
   Spin(1); 
   printf("%s\n", str); 
  } 
  return 0; 
 }
```

该程序就是读取输入，并一直循环输出读到的输入，直到输入Ctrl+C来强制终止该程序。

我们将该文件保存为cpu.c文件，并在一个单CPU的机器上编译并运行它，当然现在不太好找单核的CPU了。没关系，我们主要是为了理解其原理。

```c
prompt> gcc -o cpu cpu.c -Wall 
prompt> ./cpu "A" 
A
A
ˆC 
prompt>
```

接下来，我们同时运行同一个程序的多个不同的实例。如下：

```c
prompt> ./cpu A & ; ./cpu B & ; ./cpu C & ; ./cpu D & 
[1] 7353 
[2] 7354 
[3] 7355 
[4] 7356 
A 
B 
D 
C 
A 
B 
D 
C 
A 
C 
B 
D 
...
```

可以看到，尽管只有一个CPU，但你会感觉这4个程序在同时运行（如果是顺序运行的话，就会一直输出A，而不会输出其他的字母）

通过实例证明，在硬件的帮助下，操作系统负责将CPU转换成了无数多个虚拟CPU，从而让程序看似在同时运行。

# 虚拟化内存

截至目前，我对于虚拟化内存理解是从操作系统层面思考程序开发关于内存管理上的一种解决手段

<u>首先，从硬件角度来讲，计算机提供的内存模型就是一块大的一维的字节数组。</u>并且给每块内存分配了一个全局唯一的地址。这样在读取或写入内存时，只要指定了地址，就能知道要写入或读取内存的位置。

在多个进程同时运行的时代，每个进程都需要将自己的指令（程序）和数据加载到内存中。如果内存的一个位置被一个进程存储了数据，那么就不能再存储其他进程的内容了。其他进程的指令和数据就需要存储到内存的其他位置。

如果让每个软件自己选择可存储的内存位置，那么软件开发就会变的很复杂。那么，为了让应用软件开发变的简单，找寻可存储内存位置的工作就交给了操作系统，让操作系统来管理内存，并分配给应用进程。这就是要虚拟化内存的目的。

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202401081635527.png)​

同样，下面也通过一个代码实例来演示了操作系统对内存进行虚拟化后的效果。通过运行该程序的一个实例和同时运行多个实例，发现每个实例打印出来的内存地址都是一样的，这说明操作系统对内存进行了抽象，每个进程都有自己独立的内存，并且地址空间都是一样的。只不过这是虚拟的地址空间，实际存储的位置是由操作系统经过转换后映射到了物理内存上。

```c
#include <unistd.h>  // 包含Unix标准函数声明
#include <stdio.h>   // 包含标准输入输出函数声明
#include <stdlib.h>  // 包含内存分配函数声明
#include "common.h"  // 包含自定义头文件

int main(int argc, char *argv[]) { // 主函数开始
    int *p = malloc(sizeof(int)); // 在堆上分配一个整数大小的内存，并将地址赋给指针变量p
    assert(p != NULL); // 确保内存分配成功

    printf("(%d) memory address of p: %08x\n", getpid(), (unsigned) p); // 打印进程ID和指针p所指向内存的地址

    *p = 0; // 将指针p所指向的内存地址的值设置为0

    while (1) { // 进入无限循环
        Spin(1); // 调用Spin函数（可能是在common.h中定义），产生1秒延迟

        *p = *p + 1; // 将指针p所指向的整数值加1
        printf("(%d) p: %d\n", getpid(), *p); // 打印进程ID以及指针p所指向的整数值
    }

    return 0; // 返回0表示程序正常结束
}

```

# 并发

并发问题的发生有两个条件：<span style="font-weight: bold;" data-type="strong">同时运行多个进程或线程</span>以及这些进程或线程需要<span style="font-weight: bold;" data-type="strong">共享相同的资源</span>（比如内存）

并发问题最早出现的地方就是在操作系统上，因为操作系统上要同时运行多个程序，但硬件资源是共享的。同时也存在于多线程程序中，多线程共享所属父进程的各种资源。

并发是为了提高软件运行的效率，同时软件也必须要解决并发产生的问题。

下面是一个并发时出现问题的例子。在这个例子中，通过多线程同时对一个共享变量进行更新操作而产生了并发问题。最常见的并发例子就是多线程同时自行`i++`​的示例，这里就不再过多介绍了。

# 持久化

因为内存中的数据是临时存在的。如果断电或系统崩溃，内存中的数据就会丢失。所以，我们需要在硬件和软件的共同作用下，将数据持久化存储。

众所周知，用于持久化数据的设备是磁盘。而在操作系统中管理磁盘的软件叫做文件系统（file system）。文件系统负责以高效和可靠的方式，将用户创建的任何文件存储在系统的磁盘上。这里注意一点，操作系统中，存储和管理信息的基本单位是文件。

<span style="font-weight: bold;" data-type="strong">在持久化主题中，要研究的两方面的内容：一方面操作系统要提供给应用软件哪些API接口，以便应用软件可以完成创建文件、写入文件以及关闭文件等类似的操作。另一方面，操作系统还要知道将写入的数据存在磁盘的什么位置，然后还需要按照文件系统所维护的各种结构来记录本次数据的相关信息</span>。

当然，还有性能方面的考虑，比如延迟写入磁盘，批量写入磁盘以及优化文件系统的数据结构等。

# 一图看懂历史

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202401081635969.png)​
