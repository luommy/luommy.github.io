---
title: Golang测试
date: '2023-11-22 09:53:54'
updated: '2023-11-22 15:14:58'
excerpt: 记录关于golang的测试相关内容，如单测、基测等
categories:
  - Golang
  - 重学系列
permalink: /post/golang-test-related-mbnma.html
comments: true
toc: true
---

# Golang测试相关

# 引言

Go 语言的单元测试默认采用*官方自带的测试框架*，通过引入 testing 包，通过执行 `go test`​ 命令来实现单元测试功能。

在源代码包目录内，所有以 `_test.go`​ 为后缀名的源文件会被 `go test`​ 认定为单元测试的文件，这些单元测试的文件不会包含在 `go build`​ 的源代码构建中，而是单独通过 go test 来编译并执行。

不写测试的开发不是好程序员。TDD（Test Driven Development）在国内其实都不太关注测试这一部分。

这篇文章主要介绍下在Golang中如何做单元测试和基准测试，同时自己测试时也发现了一些问题。

# 单元测试

## **Go 单元测试的基本规范**

Go 单元测试的基本规范如下：

* 每个测试函数都必须导入 testing 包。测试函数的命名类似`func TestName(t *testing.T)`​，入参必须是 `*testing.T`​类型
* 测试函数的函数名必须以大写的 Test 开头，**后面紧跟函数名**，要么是大写，要么就是下划线，比如 `func TestName(t *testing.T)`​ 或者 `func Test_name(t *testing.T)`​ 都是可以的， 但是 `func Testname(t *testing.T)`​这不会被检测到，一定要注意
* 通常情况下，需要将**测试文件和源代码放在同一个包内**。一般测试文件的命名，都是 `{source_filename}_test.go`​，比如源代码文件是ai.go ，那么就会在 ai.go 的相同目录下，再建立一个 ai_test.go 的单元测试文件去测试 ai.go 文件里的相关方法。

当运行 go test 命令时，go test 会遍历当前目录下所有的 `*_test.go`​ 中符合上述命名规则的函数，然后生成一个临时的 main 包用于调用相应的测试函数，然后构建并运行、报告测试结果，最后清理测试中生成的临时文件。

## 常用命令

***单元测试：***

* ​`go test `​
* ​`go test -v`​
* ​`go test -v -run="方法名关键字"`​

***覆盖率测试：***

* ​`go test -cover -coverprofile=输出文件名字`​
* ​`go tool cover -html=输出文件名字`​

-v参数: 是在测试结果中补充被测函数名称和运行时间

-run参数: 它对应一个正则表达式，只有函数名匹配上的测试函数才会被`go test`​命令执行。

## 测试示例

这里参考七米的示例：

定义一个`split`​的包，包中定义了一个`Split`​函数，函数功能是为了从一个s字符串中去除sep，具体实现如下：

```go
// split/split.go

package split

import "strings"

// split package with a single split function.

// Split slices s into all substrings separated by sep and
// returns a slice of the substrings between those separators.
func Split(s, sep string) (result []string) {
	i := strings.Index(s, sep)

	for i > -1 {
		result = append(result, s[:i])
		s = s[i+1:]  //这里有bug，没有考虑到sep为多个字符的情况：s = s[i+len(sep):] 这里使用len(sep)获取sep的长度
		i = strings.Index(s, sep)
	}
	result = append(result, s)
	return
}
```

然后 ***在相同的目录下 *** 写测试函数（可以写多个测试函数，运行时可加run参数正则匹配对应的被测函数，如果不加参数默认测试全部的）：

```go
// split/split_test.go

package split

import (
	"reflect"
	"testing"
)

// 这里注意函数名规范：Test+首字母大写的被测函数名
func TestSplit(t *testing.T) { // 测试函数名必须以Test开头，必须接收一个*testing.T类型参数
	got := Split("a:b:c", ":")         // 程序输出的结果
	want := []string{"a", "b", "c"}    // 期望的结果
	if !reflect.DeepEqual(want, got) { // 因为slice不能比较直接，借助反射包中的方法比较
		t.Errorf("expected:%v, got:%v", want, got) // 测试失败输出错误提示
	}
}
```

此时的目录结构：

```go
split $ ls -l
total 16
-rw-r--r--  1 liwenzhou  staff  408  4 29 15:50 split.go
-rw-r--r--  1 liwenzhou  staff  466  4 29 16:04 split_test.go
```

然后在此目录下执行`go test`​，

结果如下：

```go
split $ go test
PASS
ok      github.com/Q1mi/studygo/code_demo/test_demo/split       0.005s
```

PASS标志测试成功

一般都习惯加下 -v 参数

再写一个测试函数TestMoreSplit，具体内容已省略，失败的情况：

```go
split $ go test -v
=<span style="font-weight: bold;" class="mark"> RUN   TestSplit
--- PASS: TestSplit (0.00s)
</span>= RUN   TestMoreSplit
--- FAIL: TestMoreSplit (0.00s)
    split_test.go:21: expected:[a d], got:[a cd]
FAIL
exit status 1
FAIL    github.com/Q1mi/studygo/code_demo/test_demo/split       0.005s
```

## 测试用例组

这实际环境下通常要考虑一些更全面的东西，比如中文切割。

参考七米的例子：有多个测试用例组

```go
func TestSplit(t *testing.T) {
   // 定义一个测试用例类型
	type test struct {
		input string
		sep   string
		want  []string
	}
	// 定义一个存储测试用例的切片
	tests := []test{
		{input: "a:b:c", sep: ":", want: []string{"a", "b", "c"}},
		{input: "a:b:c", sep: ",", want: []string{"a:b:c"}},
		{input: "abcd", sep: "bc", want: []string{"a", "d"}},
		{input: "沙河有沙又有河", sep: "沙", want: []string{"河有", "又有河"}},
	}
	// 遍历切片，逐一执行测试用例
	for _, tc := range tests {
		got := Split(tc.input, tc.sep)
		if !reflect.DeepEqual(got, tc.want) {
			t.Errorf("expected:%v, got:%v", tc.want, got)
		}
	}
}
```

go test 执行后，发现并不能直观的看出

```go
split $ go test -v
=<span style="font-weight: bold;" class="mark"> RUN   TestSplit
--- FAIL: TestSplit (0.00s)
    split_test.go:42: expected:[河有 又有河], got:[ 河有 又有河]
FAIL
exit status 1
FAIL    github.com/Q1mi/studygo/code_demo/test_demo/split       0.006s
```

这里有一个小技巧：使用​`%#v`​的格式化方式

t.Errorf("expected:%v, got:%#v", tc.want, got)

然后就很直观了：

```go
split $ go test -v
</span>= RUN   TestSplit
--- FAIL: TestSplit (0.00s)
    split_test.go:42: expected:[]string{"河有", "又有河"}, got:[]string{"", "河有", "又有河"}
FAIL
exit status 1
FAIL    github.com/Q1mi/studygo/code_demo/test_demo/split       0.006s
```

然后就再优化下被测函数或者干脆直接把测试函数的期望改了

改测试函数期望：`want: []string{"","河有", "又有河"}`​

改被测函数：比如在`result = append(result, s)`​之前判断下len(s)是否大于0,或者考虑下strings.Index的合理性...

## 子测试

如果在测试中有大量的测试用例，不好区分哪个测试用例失效了，这种情况下要用到子测试，其实也可以用map的方式

map：

```go
func TestSplit(t *testing.T) {
	type test struct { // 定义test结构体
		input string
		sep   string
		want  []string
	}
	tests := map[string]test{ // 测试用例使用map存储
		"simple":      {input: "a:b:c", sep: ":", want: []string{"a", "b", "c"}},
		"wrong sep":   {input: "a:b:c", sep: ",", want: []string{"a:b:c"}},
		"more sep":    {input: "abcd", sep: "bc", want: []string{"a", "d"}},
		"leading sep": {input: "沙河有沙又有河", sep: "沙", want: []string{"河有", "又有河"}},
	}
	for name, tc := range tests {
		got := Split(tc.input, tc.sep)
		if !reflect.DeepEqual(got, tc.want) {
			t.Errorf("name:%s expected:%#v, got:%#v", name, tc.want, got) // 将测试用例的name格式化输出
		}
	}
}
```

子测试：t.Run()

```go
func TestSplit(t *testing.T) {
	type test struct { // 定义test结构体
		input string
		sep   string
		want  []string
	}
	tests := map[string]test{ // 测试用例使用map存储
		"simple":      {input: "a:b:c", sep: ":", want: []string{"a", "b", "c"}},
		"wrong sep":   {input: "a:b:c", sep: ",", want: []string{"a:b:c"}},
		"more sep":    {input: "abcd", sep: "bc", want: []string{"a", "d"}},
		"leading sep": {input: "沙河有沙又有河", sep: "沙", want: []string{"河有", "又有河"}},
	}
	for name, tc := range tests {
		t.Run(name, func(t *testing.T) { // 使用t.Run()执行子测试
			got := Split(tc.input, tc.sep)
			if !reflect.DeepEqual(got, tc.want) {
				t.Errorf("expected:%#v, got:%#v", tc.want, got)
			}
		})
	}
}
```

再go test：

```go
split $ go test -v
=<span style="font-weight: bold;" class="mark"> RUN   TestSplit
</span>= RUN   TestSplit/leading_sep
=<span style="font-weight: bold;" class="mark"> RUN   TestSplit/simple
</span>= RUN   TestSplit/wrong_sep
=== RUN   TestSplit/more_sep
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/leading_sep (0.00s)
        split_test.go:83: expected:[]string{"河有", "又有河"}, got:[]string{"", "河有", "又有河"}
    --- PASS: TestSplit/simple (0.00s)
    --- PASS: TestSplit/wrong_sep (0.00s)
    --- PASS: TestSplit/more_sep (0.00s)
FAIL
exit status 1
FAIL    github.com/Q1mi/studygo/code_demo/test_demo/split       0.006s
```

补充：

> 可以通过`-run=RegExp`​来指定运行的测试用例，还可以通过`/`​来指定要运行的子测试用例，例如：`go test -v -run=Split/simple`​只会运行`simple`​对应的子测试用例。

## 意料之外的情况

将被测函数NewLRU与测试函数NewLRU_test的目录下有其他的文件，且这些有文件在main包中，这种情况下，main会被检测到，导致其它不相干的函数也被检测到了。

如果目录像我这种：我有一个LRU算法函数和一个测试LRU的测试函数

​![6544c0afd9b1f0b5459a30becc26ce3](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311221516056.png)​

直接go test会报错：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311221516907.png)​

***<u>解决方式：</u>***

1. 执行文件：`go test .\NewLRU.go .\NewLRU_test.go -v`​ ,可这也太麻烦了
2. 移到的新包下，比如新建个test目录，将`NewLRU.go`​与`NewLRU_test.go`​一进去

通常业界也观察到，大部分测试文件和被测文件是放在同一个目录下的

# 覆盖率测试

测试覆盖率是你的代码被测试套件覆盖的百分比。通常我们使用的都是语句的覆盖率，也就是在测试中至少被运行一次的代码占总代码的比例。

Go提供内置功能来检查你的代码覆盖率。我们可以使用`go test -cover`​来查看测试覆盖率。例如：

```go
PASS
coverage: 100.0% of statements
ok      main/test       0.818s
```

o还提供了一个额外的`-coverprofile`​参数，用来将覆盖率相关的记录信息输出到一个文件。例如：

​`go test -cover -coverprofile=c`​

执行`go tool cover -html=c`​，使用`cover`​工具来处理生成的记录信息，该命令会打开本地的浏览器窗口生成一个HTML报告

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311221516624.png)​

每个用绿色标记的语句块表示被覆盖了，而红色的表示没有被覆盖。

# 基准测试

> 我一般理解为性能测试，在一定工作负载下检测程序性能的一种方法。
>
> 后续补充.....
