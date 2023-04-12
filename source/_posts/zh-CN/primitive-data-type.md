---
title: Java_SE_第二讲：原生数据类型
updated: 2022-10-10 19:49:40
excerpt: windows_notepadeditplusultraeditgvimlinux_vivimgeditjava中的数据类型分为两大类_）原生数据类型（primitivedatatype））引用类型（对象类型）（referencetype）变量与常量_所谓常量就是值不会变化的量_所谓变量就是值可以变化的量。如何定义变量？变量类型变量名_inta_如何为变量赋值？变量名=变量值_a=_=表示赋值将等号右边的值赋给了左边的变量。java中使用==表示相等等价于数学中的=。综合变量定义与赋值。变量类型变量名_变
tags:
  - 注释
  - 变量
  - 变量名
  - 表示
  - 类型
  - javase
  - datatype
  - primary
categories:
  - JavaSE
  - 后端开发
permalink: /post/primitive-data-type.html
comments: true
toc: true
---
1. Windows: notepad, editplus, ultraedit, gvim   Linux: vi, vim, gedit
2. Java 中的数据类型分为两大类：

    1）原生数据类型 （Primitive Data Type）

    2）引用类型（对象类型） （Reference Type）
3. 变量与常量：所谓常量，就是值不会变化的量；所谓变量，就是值可以变化的量。
4. 如何定义变量？   

    变量类型 变量名;   int a;
5. 如何为变量赋值？  
    变量名 = 变量值;  
    a = 2;  
    = 表示赋值，将等号右边的值赋给了左边的变量。  
    Java 中使用==表示相等，等价于数学中的=。
6. 综合变量定义与赋值。  
    变量类型 变量名;  
    变量名 = 变量值;  
    int a;  
    a = 1;  
    可以将上面两个步骤合二为一：  
    变量类型 变量名 = 变量值;  
    int a = 1;
7. 变量名：在 Java 中，变量名以下划线、字母、`$` 符号开头，并且后跟下划线、字母、`$` 符号以及数字。总之，Java 中的变量名不能以数字开头。
8. Java 中的原生数据类型共有 8 种：  
    1）整型：使用 int 表示。（32 位）  
    2）字节型：使用 byte 表示。（表示-128～127 之间的 256 个整数）。  
    3）短整型：使用 short 表示。（16 位）  
    4）长整型：使用 long 表示。（64 位）
9. 关于计算机系统中的数据表示 8位：bit（只有 0，1 两种状态），是计算机系统中的最小数据表示单位。

    字节：byte， 

    1 byte = 8 bit。   

    1 KB = 1024 Byte （1Kg = 1000g，与计算机系统不同）   

    1 MB = 1024 KB   1 GB = 1024 MB
10. 注释。注释是给人看的，不是给计算机看的。

     Java 中共有 3 种类型的注释：   

     1）单行注释：以//开头，//后面的所有内容均被当作注释处理。  

     2）多行注释：以 /* 开头，以 */ 结束，中间的所有内容均被当作注释处理。多行注释来源于  
     C/C++。关于多行注释，需要注意的是，多行注释不能嵌套。   

     3）另一种多行注释。用于产生 Java Doc 帮助文档。暂且不介绍。

> 文章更新历史
>
> 2022/05/08 fix:修改备注。