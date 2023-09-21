---
title: Reverse a String
date: '2023-09-22 00:21:13'
updated: '2023-09-22 00:26:39'
excerpt: 将含有字母、数字、中文的混合字符串反转
categories:
  - 算法加练
permalink: /post/string-reverse-1jimhs.html
comments: true
toc: true
---
# 字符串反转

翻转含有`中文、数字、英文字母`​的字符串

如：`"子asdf黑g白hjkl小"`​

```go
package main

import "fmt"

/**
   @Title main
   @Description: 1.rune关键字，从golang源码中看出，它是int32的别名（-2^31 ~ 2^31-1），比起byte（-128～127），可表示更多的字符。
		 2.由于rune可表示的范围更大，所以能处理一切字符，当然也包括中文字符。在平时计算中文字符，可用rune。
		 3.因此将字符串转为rune的切片，再进行翻转，完美解决。
   @Author luommy 2023-09-22 00:19:14
**/
func main() {
	src := "子asdf黑g白hjkl小"
	// int32 is the set of all signed 32-bit integers. Range: -2147483648 through 2147483647.
	str := reverse([]rune(src))
	fmt.Printf("%v\n", string(str))
}

/**
   @Title reverse
   @Description
   @Author luommy 2023-09-22 00:12:29
   @Param s
   @Return []rune
**/
func reverse(s []rune) []rune {
	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
		s[i], s[j] = s[j], s[i]
	}
	return s
}

```

‍

运行结果：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202309220031674.png)​
