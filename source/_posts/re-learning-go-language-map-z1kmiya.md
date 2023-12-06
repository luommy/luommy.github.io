---
title: 重学Go语言 | Map
date: '2023-12-05 15:35:49'
updated: '2023-12-06 17:48:36'
excerpt: 重温Map基础
categories:
  - Golang
  - 重学系列
permalink: /post/re-learning-go-language-map-z1kmiya.html
comments: true
toc: true
tags:
  - golang
---

# 重学Go语言 | Map

> Go重学系列不追求底层和深度，只求重温下没有使用过或者使用过的一些操作，要注意的细节，以及彻底掌握的一种前提思路。后续讲重学与底层部分练习在一起进行双链操作。
>
> 重学篇在自己重温Go的同时，也希望努力做最好的初学教程
>
> 本篇思路：什么是map、map的格式与数据类型限制、map的特征、如何创建与初始化等

# Map简述

```
Go语言中的map(映射、字典、哈希表)是一种内置的数据结构，它是一个无序的key-value对的集合，
比如以身份证号作为唯一键来标识一个人的信息。Go语言中并没有提供一个set类型，
但是map中的key也是不相同的，可以用map实现类似set的功能。
```

从表面上看map大致是这样的：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312061138052.png)​

​![](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312061138650.png)​

从底层看：

​`map`​是一个无序的键值对(`key-value`​)集合，其底层数据结构是一个哈希表，通过<span style="font-weight: bold;" data-type="strong">哈希函数</span>，将`key`​转换为对哈希表中的索引，将`value`​存储到索引对应的位置，在`map`​中查找、删除、查找value的时间复杂度`O(1)`​。

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312061138827.png)​

# map格式与数据类型限制

<span style="font-weight: bold;" data-type="strong">关键点：key数据类型限制</span>

map的value可以是Go支持的任意数据类型，而key则所有限制：

> <span style="font-weight: bold;" data-type="strong">key的数据类型必须是可以使用</span>​ <span style="font-weight: bold;" data-type="strong">`=`</span> ​<span style="font-weight: bold;" data-type="strong">和</span>​ <span style="font-weight: bold;" data-type="strong">`!=`</span> ​<span style="font-weight: bold;" data-type="strong">进行比较</span>
>
> <u>所以，key不能是函数、切片、map，因此这些数据类型不能进行比较，另外，而数组和结构体则可以作为map的key，不过，如果数组的元素包含函数、切片、map，则数组不能作为map的key，结构体的字段如果有以上三者，也同样不能作为map的key。</u>

```go
type Test struct {
 ID   string
 Name string
}

//正确
m := map[Test]int{}

type Test struct {
 ID     string
 Name   string
  scores []int
}

//报错
m := map[Test]int{}
```

<span style="font-weight: bold;" data-type="strong">key的值必须是唯一，同一个map中不能相同的两个key</span>

```go
m := map[string]string{
 "name":"小明",
 "age":"24",
 "name":"小墨"//报错，不能有相同的key
}
```

```Go
//在Go语言中，用 map[KeyType]ValueType 表示一个map，
//其中，KeyType 表示 key 的数据类型，ValueType 表示 value 的数据类型：
map[keyType]valueType
```

```go
// ※ 重点 ※  
// 在一个 map 里所有的键都是唯一的，而且必须是支持 == 和 != 操作符的类型，
// 切片、函数以及包含切片的结构类型这些类型由于具有引用语义，不能作为映射的键，
// 使用这些类型会造成编译错误：
dict := map[ []string ]int{} //err, invalid map key type []string
```

---✩ ✰ ✪ ✫---

map值可以是任意类型，没有限制。map里所有键的数据类型必须是相同的，值也必须如此，但键和值的数据类型可以不相同。

<span style="font-weight: bold;" data-type="strong">注意：</span>​<u>map是无序的，我们无法决定它的返回顺序，所以，每次打印结果的顺序有可能不同。</u>

--&gt;这个地方就有深度了，为什么map遍历是无序的？ Java中的set也是如此吗？为什么？原理是否一样？

# map的特征

推导总结：

* map是无序的
* map的key是唯一的
* map是引用数据类型，因此在使用前必须初始化
* 函数，切片，map等数据类型不能作为map的key。key必须是可比较的类型，可以是结构体和数组，不能是切片、函数、map，且结构体和数组中也不能包含以上三者中的任何

# 创建及初始化Map

很多教程都是直接告诉你如何初始化，可能只教一种实现方式，不教所以然，这样很片面

## 未经初始化

由于`map`​是引用数据类型，其底层引用一个哈希表，因此未经初始化(即未分配到内存空间)的`map`​无法直接使用：

```go
var m map[string]int

m["a"] = 1 
//未初始化，报错
```

## 初始化方式

初始化map有两种方式：

* 使用make函数
* 字面量初始化

[刘丹冰：New与Make](https://www.yuque.com/aceld/golang/xu2t51)

理论上New也是可行的，但是十分不推荐

### make函数

内置函数make可以为`map`​类型的变量分配内存：

```go
m := make(map[string]string)
m["name"] = "test"
m["age"] = "12"
```

此外还可以指定容量初始化：

这种方式有好处也有劣势，好处是省去了频繁地扩容，坏处是如果一次性建立很大的容量但实际上并不需要会造成资源浪费

```go
m := make(map[int]string, 10) 	//第2个参数指定容量“10”
fmt.Println(m)                	//map[]
```

这种方法指定了map的初始创建容量。与slice类似，后期在使用过程中，map可以自动扩容。

只不过map更方便一些，不用借助类似append的函数，直接赋值即可。如，m[17] = "Nami"。赋值过程中，key如果与已有map中key重复，会将原有map中key对应的value覆盖。但是！对于map而言，可以使用len()函数，<span style="font-weight: bold;" data-type="strong">*<u>但不能使用 cap() 函数。</u></span>*

map是一种可以动态增长的数据结构，但由于其底层实现的复杂性，能直接获取其容量，只能使用`len()`​函数来获取其元素数量。

### 字面量初始化

1. 初始化同时赋值

通过字面量初始化`map`​时，可以给`map`​中的`key`​和`value`​赋初始值：

```go
// 这样直接指定初值，要保证key不重复。
// 注意！这是常用的方式，使用“：”自动推导的方式
m := map[string]string{
 "name":"test",
 "age":"12",
}

// 当然了，还有一种不常用的方式：
// 定义的同时完成初始化，但有一说一这种方式太不简洁了
var m1 map[int]string = map[int]string{1: "Luffy", 2: "Sanji"}
    fmt.Println(m1) //map[1:Luffy 2:Sanji]
```

2. 只初始化，不赋值

<span style="font-weight: bold;" data-type="strong">常用：</span> 

如果不想给map初始化数据，也可以声明一个空的map类型，自动分配了内存：

```go
// 这里的`m`是一个字符串键和字符串值类型的map，但你也可以根据需要更改其键和值类型。
m := map[string]string{}
```

> 注意，空的map已经初始化好了，只是没有存入值而已，而直接声明一个map变量时，该map变量为nil，这两者不一样

<span style="font-weight: bold;" data-type="strong">不常用：</span> 

声明一个未初始化的map的方式如下：

```go
var m map[string]string
```

这里的`m`​是一个字符串键和字符串值类型的map变量，但它没有被初始化，因此不能直接使用，如果你尝试在其上执行读或写操作，会触发panic错误。要使用未初始化的map，还需要使用make函数对其进行初始化，如下所示：

```go
// var这种方式也不常用，因为还要再写这一行完成内存分配才能使用
m = make(map[string]string)
```

这样，你的map就可以安全地使用了。

### 非要用new初始化

```go
package main

import "fmt"

func main() {
    // 使用 new 创建 map
    myMap := new(map[string]int)
  
    // 初始化 map
    *myMap = make(map[string]int)
  
    // 向 map 中添加键值对
    (*myMap)["apple"] = 1
    (*myMap)["banana"] = 2
    (*myMap)["orange"] = 3
  
    // 遍历 map 并打印键值对
    for key, value := range *myMap {
        fmt.Println(key, ":", value)
    }
}
```

‍

# 常用操作

## 赋值与访问

### 赋值

赋值上来说：有直接初始化的同时直接赋值，也有先初始化，再赋值这两种

```Go
    m1 := map[int]string{1: "Luffy", 2: "Sanji"}
    m1[1] = "Nami"   //修改
    m1[3] = "Zoro"  //追加， go底层会自动为map分配空间
    fmt.Println(m1) //map[1:Nami 2:Sanji 3:Zoro]

    m2 := make(map[int]string, 10) 	//创建map，指定容量
    m2[0] = "aaa"
    m2[1] = "bbb"
    fmt.Println(m2)           		//map[0:aaa 1:bbb]
    fmt.Println(m2[0], m2[1]) 		//aaa bbb
```

### 访问

通过key，可以访问map变量中的value：

```go
rank : = map[string]int{
  "PHP":90,
  "Go":99
  "Java":95
}

fmt.Println(rank["PHP"])
```

*如果对应的key不存在，则会返回对应value数据类型的空值，比如value为string，则返回空字符串：*

```go
//因为value为int
fmt.Println(rank["Python"]) 
// 输出：0

m := map[string]string{"name":"小张"}

//由于value为string类型
fmt.Println(m["age"]) 
//输出空字符串
```

## 判断key是否存在

如果我们想在通过key访问map之前就确定对应的key是否存在，有另外一种写法：

```go
v, ok := m[k]
```

上面的表达式中，有第二个返回值`ok`​，该值为boolean类型，当key存在时，`ok`​的值为true，`v`​为对应的`value`​；否则为`ok`​为`false`​,`v`​为空值。

```go
// 方法一
v,ok := rank[k]
if ok {
 fmt.Println(v)
}

// 方法二，这种更常见
if v,ok := rank[k];ok{
  fmt.Println(v)
}
```

有时候可能需要知道对应的元素是否真的是在map之中。  
可以使用下标语法判断某个key是否存在。  
map的下标语法将产生两个值，其中第二个是一个布尔值，用于报告元素是否真的存在。  
如果key存在，第一个返回值返回value的值。第二个返回值为 true。

```go
// 方法一：
v,ok := rank[k]
if ok {
 fmt.Println(v)
}

// 方法二：这种可能更常见
if v,ok := rank[k];ok{
  fmt.Println(v)
}
```

## 遍历

使用`for-range`​语句可以遍历`map`​，获得`map`​的`key`​和`value`​：

```go
m := map[string]string{
    "name":"xiaoming",
    "age":"18岁"
}
for k,v := range m{
  fmt.Println(k,v)
}
```

注意：

Map的迭代顺序是不确定的，并且不同的[哈希函数]实现可能导致不同的遍历顺序。  
在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同。  
这是故意的，每次都使用随机的遍历顺序可以强制要求程序不会依赖具体的哈希函数实现。

```Go
m1 := map[int]string{1: "Luffy", 2: "Sanji"}
//遍历1，第一个返回值是key，第二个返回值是value
for k, v := range m1 {
        fmt.Printf("%d ----> %s\n", k, v)
//1 ----> Luffy
//2 ----> yoyo
    }

//遍历2，第一个返回值是key，第二个返回值是value（可省略）
for k := range m1 {
        fmt.Printf("%d ----> %s\n", k, m1[k])
//1 ----> Luffy
//2 ----> Sanji
    }
```

## 删除

要删除map的value，可以使用Go内置的`delete`​函数，该函数格式如下：

```go
func delete(m map[KeyType]ValueType, key Type)
```

该函数的第一个参数是我们要操作的map类型的变量，第二个参数表示要删除哪个key：

```go
m := map[string]int{
  "a":1,
  "b":2
}

fmt.Println(m)
delete(m,"a")
fmt.Println(m)
```

使用delete()函数，指定key值可以方便的删除map中的k-v映射。

```Go
m1 := map[int]string{1: "Luffy", 2: "Sanji", 3: "Zoro"}

for k, v := range m1 {	//遍历，第一个返回值是key，第二个返回值是value
        fmt.Printf("%d ----> %s\n", k, v)
    }
//1 ----> Sanji
//2 ----> Sanji
//3 ----> Zoro
delete(m1, 2) 		//删除key值为2的map

for k, v := range m1 {
        fmt.Printf("%d ----> %s\n", k, v)
    }
//1 ----> Luffy
//3 ----> Zoro
 
```

```Go
// 使用delete删除一个不存在的key
delete(m1, 5) 		//删除key值为5的map

for k, v := range m1 {
        fmt.Printf("%d ----> %s\n", k, v)
    }
//1 ----> Luffy
//3 ----> Zoro

```

Map输出结果依然是原来的样子，且不会有任何错误提示。

delete()操作是安全的，即使元素不在map中也没有关系；如果查找删除失败将返回value类型对应的零值。

## 获取长度

map中无法获取容量，也就是无法使用cap函数，它的容量是动态变化的，内部是通过哈希表实现的，底层实现与数组、切片不同；同时，他们的扩容机制【slice与map】有何不同？

要获取map的长度，同样是用内置的`len`​函数：

```go
var user = map[string]string{
 "id":   "0001",
 "name": "小张",
 "age":  "18岁",
}
fmt.Println(len(user)) 
//输出：3
```

## Map嵌套

由于`map`​的`value`​并没有数据类型的限制，所以`value`​也可以是另一个`map`​类型：

> 理论上map嵌套map可以一直嵌套下去，但一般不会这么做的。很丑！

```go
mm := map[string]map[int]string{
	"a": {1: "test1"},
	"b": {2: "test2"},
	"c": {2: "test3"},
}
fmt.Println(mm)
```

## Map做函数参数

与slice 相似，在函数间传递映射并不会制造出该映射的一个副本，不是[值传递]，而是引用传递

```Go
func DeleteMap(m map[int]string, key int) {
	delete(m, key) //删除key值为2的map
	for k, v := range m {
        fmt.Printf("len(m)=%d, %d ----> %s\n", len(m), k, v)
}
//len(m)=2, 1 ----> Luffy
//len(m)=2, 3 ----> Zoro
}

func main() {
    m := map[int]string{1: "Luffy", 2: "Sanji", 3: "Zoro"}
    DeleteMap(m, 2) 	//删除key值为2的map

	for k, v := range m {
        fmt.Printf("len(m)=%d, %d ----> %s\n", len(m), k, v)
}
//len(m)=2, 1 ----> Luffy
//len(m)=2, 3 ----> Zoro
}
```

## Map做函数返回值

返回的依然是引用

```Go
func test() map[int]string {
//  m1 := map[int]string{1: "Luffy", 2: "Sanji", 3: "Zoro"}
    m1 := make(map[int]string, 1)     // 创建一个初创容量为1的map
    m1[1] = "Luffy"
    m1[2] = "Sanji"                   // 自动扩容
    m1[67] = "Zoro"
    m1[2] = "Nami"                	  // 覆盖 key值为2 的map
    fmt.Println("m1 = ", m1)
	return m1
}

func main() {
    m2 := test()                  	  // 返回值 —— 传引用
    fmt.Println("m2 = ", m2)
}
```

输出：

```bash
m1 =  map[1:Luffy 2:Nami 67:Zoro]
m2 =  map[2:Nami 67:Zoro 1:Luffy]
```

## Map的排序

​`map`​是无序的，所以每次遍历`map`​输出的顺序都不一定相同

```go
var user = map[string]string{
 "id":   "0001",
 "name": "小张",
 "age":  "18岁",
}
for k, v := range user {
 fmt.Println(k, ":", v)
}
for k, v := range user {
 fmt.Println(k, ":",v)
}
```

要有序地遍历一个map类型的变量，可以这么做：

利用切片将map中的key固定排序，这样就稳定了顺序

```go
order := []string{}
for k, _ := range user {
 order = append(order, k)
}

// 对user中的key进行升序排序
// sort.Strings(order)

for _, v := range order {
 fmt.Println(user[v])
}
for _, v := range order {
 fmt.Println(user[v])
}
```

## Map之间无法比较，只能与nil比较

​`map`​类型变量之间不能进行比较，`map`​只能与`nil`​进行比较：

```go
var m map[int]string
//判断是否等于nil
if m == nil{
 fmt.Println("m hasn't been initialized")
}

m1 := map[string]string{"name": "小明"}
m2 := map[string]string{"name": "小明"}

//报错
if m1 == m2 {
 fmt.Println("相等")
}
```

这个地方我看到有的教程说：“nil map 和空 map 是相等的，只是 nil map 不能添加元素。”这个map之间不能被比较何来相等一说？对此有疑问

## 不能对Map的value进行取址操作

取址操作作用：

当对数组和切片进行取址操作时，我们可以实现参数传递、修改元素和实现数据结构等功能。以下是一些示例：

1. 参数传递：

```go
package main

import "fmt"

func modifyElement(arr *[3]int) {
    (*arr)[0] = 100
}

func main() {
    array := [3]int{1, 2, 3}
    modifyElement(&array)
    fmt.Println(array) // 输出 [100 2 3]
}
```

在这个例子中，通过对数组元素取址，将数组作为指针传递给函数`modifyElement`​，在函数内部可以直接修改数组元素的值。

2. 修改元素：

```go
package main

import "fmt"

func main() {
    slice := []int{1, 2, 3}
    fmt.Println(slice) // 输出 [1 2 3]
    slicePtr := &slice
    (*slicePtr)[0] = 100
    fmt.Println(slice) // 输出 [100 2 3]
}
```

在这个例子中，通过对切片元素取址，可以直接修改切片中的元素值。

3. 实现数据结构：

```go
package main

import "fmt"

type Node struct {
    Value int
    Next  *Node
}

func main() {
    node1 := Node{Value: 1}
    node2 := Node{Value: 2}
    node1.Next = &node2

    fmt.Println(node1.Value)   // 输出 1
    fmt.Println(node1.Next.Value)  // 输出 2
}
```

在这个例子中，我们通过对结构体中的指针字段进行取址操作，实现了链表数据结构。

这些示例展示了如何使用数组和切片的取址操作来实现参数传递、修改元素和实现数据结构，从而充分展示了取址操作的用处。

Go的数组和切片允许对元素进行取址操作，但不允许对map的元素进行取址操作：

```go
//对数组元素取址
a := [3]int{1, 2, 3}
fmt.Println(&a[1])

//对切片元素取址
s := []int{1, 2, 3}
fmt.Println(&s[1])

//对map元素取址，错误
m := map[string]string{
  "test":"test",
}
fmt.Println(&m["test"])
```

<u>*为什么Go允许数组、切片进行取址操作，但要限制对map的元素取址呢？*</u> 

因为Go可以在添加新的键值对时更改键值对的内存位置。Go将在后台执行此操作，以将检索键值对的复杂性保持在恒定水平。因此，地址可能会变得无效，Go宁愿禁止访问一个可能无效的地址。

> 对map不允许直接进行取址操作的主要原因是为了避免潜在的数据竞争和安全问题。
>
> 在Go语言中，map是通过哈希表实现的，它的内部结构相对复杂。当我们对map进行取址操作时，实际上是获取了一个指向底层哈希表的指针。这样一来，如果在获取指针后对map进行了添加、删除或重新分配内存等操作，会导致指针指向的地址发生变化，从而可能使之前获取到的指针失效或指向无效的内存区域。这就会造成潜在的数据不一致性和安全隐患。
>
> 为了避免这种情况，Go语言禁止对map进行取址操作，并且在对map进行增删改等操作时，可能会触发内部的扩容和重新散列等操作，从而导致map的底层数据结构发生变化。如果允许对map进行取址操作，那么在并发访问的情况下，就可能引发竞态条件，导致数据不一致或安全问题。
>
> 相反，对数组和切片进行取址操作是允许的。这是因为数组和切片的内部结构相对简单，取址操作不会引发底层数据结构的变化。同时，数组和切片的元素是可以被直接修改的，因此可以对它们进行取址操作来实现参数传递、修改元素和实现数据结构等功能。在使用数组和切片的取址操作时，仍然需要注意并发安全和正确性问题，但相对于map，由于其内部结构的简单性，风险较小。
>
> 总而言之，在Go语言中，不允许对map进行取址操作是为了避免数据竞争和安全问题，而数组和切片则具备相对简单的内部结构和更明确的使用方式，因此允许进行取址操作。

# 补充

> 本部分有待商榷

## 什么时候需要用指针封装map

在 Go 中，大部分情况下，不需要对 map 进行指针封装。因为 map 本身是引用类型，在函数传递时传递的是指向底层数据结构的指针。这意味着在函数间传递 map 时，传递的是其引用而不是值的拷贝，因此可以直接对 map 进行操作而无需额外封装为指针。

然而，有些情况下可能需要对 map 进行指针封装：

### 1. 需要在多个函数中修改同一个 map

如果需要在多个函数中修改同一个 map 并且希望这些修改对所有函数都生效，可以使用指针封装。因为 map 是引用类型，传递指向 map 的指针可以确保在不同函数间共享相同的 map 实例，对 map 的修改可以在不同函数中体现出来。

### 2. 减少 map 的拷贝开销

在某些情况下，当 map 很大并且需要在函数之间传递时，将 map 传递为指针可以减少拷贝的开销。大的 map 传递值拷贝可能会消耗较多的内存和时间，而传递指针则只需传递地址。

### 3. 需要避免 map 自身的nil指针问题

在某些场景下，需要避免 map 本身可能出现 nil 指针的问题。通过使用指针封装 map，可以在 nil map 的情况下返回一个非 nil 的 map，避免出现空 map 操作时的错误。

当需要在多个函数中修改同一个 map 并确保修改对所有函数生效时，使用指针封装可以很有用。以下是示例：

```go
package main

import "fmt"

type MapStruct struct {
    Data map[string]int
}

func NewMapStruct() *MapStruct {
    return &MapStruct{
        Data: make(map[string]int),
    }
}

func AddData(m *MapStruct, key string, value int) {
    m.Data[key] = value
}

func main() {
    mapInstance := NewMapStruct()

    AddData(mapInstance, "Key1", 10)
    AddData(mapInstance, "Key2", 20)

    fmt.Println(mapInstance.Data) // 输出：map[Key1:10 Key2:20]
}
```

上述代码中，`AddData`​ 函数接收一个 `MapStruct`​ 的指针作为参数，并向其 map 字段中添加键值对。在 `main`​ 函数中调用 `AddData`​ 两次来修改同一个 `MapStruct`​ 实例的 map，并最终输出修改后的 map 数据。

对于第二种情况，如果需要在多个函数之间传递大型的 map，通过指针封装可以避免大的 map 被复制多次，节省内存和提高效率。以下是一个例子：

```go
package main

import "fmt"

func ProcessMap(m *map[string]int) {
    // 对传入的 map 进行处理
    (*m)["Key1"] = 100
    (*m)["Key2"] = 200
}

func main() {
    myMap := map[string]int{}

    ProcessMap(&myMap)

    fmt.Println(myMap) // 输出：map[Key1:100 Key2:200]
}
```

在这个示例中，`ProcessMap`​ 函数接收一个 `map[string]int`​ 类型的指针，并对传入的 map 进行操作。在 `main`​ 函数中调用 `ProcessMap`​ 函数，并传递了 `myMap`​ 的地址。通过指针传递，避免了对 `myMap`​ 进行复制，直接在原始的 map 上进行修改。

第三种情况，当需要避免 map 本身可能出现 nil 指针问题时，可以通过指针封装来确保 map 不会为 nil。以下是一个例子：

```go
package main

import "fmt"

type MapWrapper struct {
    Map *map[string]string
}

func NewMapWrapper() *MapWrapper {
    m := make(map[string]string)
    return &MapWrapper{Map: &m}
}

func main() {
    mapWrapper := NewMapWrapper()

    // 检查 map 是否为 nil，如果是则初始化一个空的 map
    if *(mapWrapper.Map) == nil {
        *mapWrapper.Map = make(map[string]string)
    }

    (*mapWrapper.Map)["Key1"] = "Value1"
    (*mapWrapper.Map)["Key2"] = "Value2"

    fmt.Println(*mapWrapper.Map) // 输出：map[Key1:Value1 Key2:Value2]
}
```

在这个例子中，`MapWrapper`​ 结构体包含一个指向 `map[string]string`​ 类型的指针。`NewMapWrapper`​ 函数返回一个初始化过的 `MapWrapper`​ 实例，其中的 map 指针初始化为一个空的 map。然后在 `main`​ 函数中对 map 进行操作，并确保了 map 不会是 nil。

## 关于map中的key可以是任意类型

为了说明值可以是任意类型的，这里给出了一个使用 func() int 作为值的 map——mf：

```go
package main

import "fmt"

func main() {
	mf := map[int]func() int{
		1: func() int { return 10 },
		2: func() int { return 20 },
		5: func() int { return 50 },
	}
	fmt.Println(mf)
}
```

输出结果为：map[1:0x53b9a0 2:0x53b9c0 5:0x53b9e0]

整形key的value值都被映射到了对应的函数地址

## 用slice作为map的值

既然一个 key 只能对应一个 value，而 value 又是一个原始类型，那么如果一个 key 要对应多个值怎么办？例如，当我们要处理unix机器上的所有进程，以父进程（pid 为整形）作为 key，所有的子进程（以所有子进程的 pid 组成的切片）作为value。通过将 value 定义为 []int 类型或者其他类型的切片，就可以优雅的解决这个问题。

这里有一些定义这种 map 的例子：

```go
mp1 := make(map[int][]int)
mp2 := make(map[int]*[]int)
```

示例中的 `mp1`​ 和 `mp2`​ 都是用来处理一个 key 对应多个值的情况。

* ​`mp1 := make(map[int][]int)`​：这创建了一个 map，其中每个 key 是一个整数，而每个 value 是一个整数切片。这样，对于每个 key，你可以将其值设置为包含多个整数的切片，实现了一个 key 对应多个值的存储。
* ​`mp2 := make(map[int]*[]int)`​：这个 map 与上述的 `mp1`​ 稍有不同，它的 value 是指向整数切片的指针。这意味着每个 key 对应一个指向整数切片的指针，这样可以通过指针修改对应 key 的整数切片。

举例说明：

```go
package main

import "fmt"

func main() {
    // 定义一个 map，每个 key 对应一个整数切片
    mp1 := make(map[int][]int)
  
    // 添加多个值到同一个 key 对应的切片中
    mp1[1] = append(mp1[1], 10)
    mp1[1] = append(mp1[1], 20)
  
    fmt.Println(mp1) // 输出: map[1:[10 20]]

    // 定义一个 map，每个 key 对应一个整数切片的指针
    mp2 := make(map[int]*[]int)
  
    // 添加多个值到同一个 key 对应的切片中
    slice := []int{30, 40}
    mp2[2] = &slice
  
    fmt.Println(*mp2[2]) // 输出: [30 40]
}
```

这些示例展示了如何使用这两种方式来实现一个 key 对应多个值的存储，并在需要时向切片中添加多个值。

在使用上，`map[int][]int`​ 和 `map[int]*[]int`​ 这两种方式有一些不同点：

1. <span style="font-weight: bold;" data-type="strong">数据所有权：</span>

    * ​`map[int][]int`​：在这种情况下，切片是直接存储在 map 的值中，而不是指针。这意味着每个 key 对应的值都是独立的，修改一个 key 对应的切片不会影响其他 key。
    * ​`map[int]*[]int`​：这种情况下，map 中的值是指向切片的指针。多个 key 可能共享相同的切片，因为它们都指向同一个切片。修改一个 key 对应的切片可能会影响其他 key。
2. <span style="font-weight: bold;" data-type="strong">内存管理和性能：</span>

    * ​`map[int][]int`​：由于切片是直接存储在 map 中的值，这可能会增加内存使用量，尤其当值的数量较多时。但是，访问和修改数据的性能可能更好，因为它们是直接存储在 map 中的。
    * ​`map[int]*[]int`​：这种情况下，指针存储在 map 中的值，实际的切片数据存储在堆上。它可能需要更多的内存，但在某些情况下，它可以减少复制和传递数据的开销，因为多个 key 可以共享相同的切片数据。
3. <span style="font-weight: bold;" data-type="strong">安全性：</span>

    * ​`map[int][]int`​：由于切片是直接存储在 map 中的值，对一个 key 对应的切片的修改不会影响其他 key。这可以提供更好的安全性，因为每个 key 拥有独立的数据。
    * ​`map[int]*[]int`​：当多个 key 共享同一个切片时，一个 key 的修改可能会影响其他 key，可能导致意外的数据修改。在多个地方使用同一个切片时，需要更小心地管理数据的修改。

因此，在使用时，需要根据具体的场景和需求来选择适当的方式。如果需要独立的数据副本或更好的数据安全性，`map[int][]int`​ 可能更适合；如果需要共享数据、降低内存占用或减少复制开销，`map[int]*[]int`​ 可能更适合。

这个差异的优缺点有待商榷

当使用 `map[int]*[]int`​ 的方式时，多个 key 可能共享相同的切片。因此，对一个 key 对应的切片的修改可能会影响其他 key。

举个例子：

```go
package main

import (
	"fmt"
)

func main() {
	myMap := make(map[int]*[]int)

	// 创建一个切片并将其地址存储在 map[1] 中
	slice := []int{10, 20}
	myMap[1] = &slice

	// 将 map[2] 指向 map[1] 所指向的切片
	myMap[2] = myMap[1]

	// 修改 map[1] 中的切片
	*myMap[1] = append(*myMap[1], 30)

	// 输出 map[2]，查看是否受到影响
	fmt.Println(*myMap[2]) // 输出：[10 20 30]
}
```

在这个示例中，我们创建了一个 `map[int]*[]int`​ 类型的 map，将一个切片的地址存储在 map[1] 中。接着，我们将 map[2] 的值指向了 map[1] 所指向的同一个切片。

然后，我们修改了 map[1] 中的切片，通过在 map[1] 所指向的切片中添加一个新元素。接着，我们打印了 map[2] 的值，发现其值也发生了变化，因为 map[2] 实际上指向了 map[1] 所指向的同一个切片。

这就说明了当使用 `map[int]*[]int`​ 时，一个 key 的切片修改会影响到其他 key，因为它们指向的是相同的切片。

<span style="font-weight: bold;" data-type="strong"><u>我的尝试：</u></span>

指针所指向的地址相同的情况：

```go
package main

import "fmt"

func main() {
	// 定义一个 map，每个 key 对应一个整数切片
	mp1 := make(map[int][]int)

	// 添加多个值到同一个 key 对应的切片中
	mp1[1] = append(mp1[1], 10)
	mp1[1] = append(mp1[1], 20)

	fmt.Println(mp1) // 输出: map[1:[10 20]]

	// 定义一个 map，每个 key 对应一个整数切片的指针
	mp2 := make(map[int]*[]int)

	// 添加多个值到同一个 key 对应的切片中
	slice1 := []int{30, 40}
	mp2[2] = &slice1

	mp2[3] = mp2[2]
	fmt.Println(*mp2[2])
	fmt.Println(*mp2[3])
	*mp2[2] = append(*mp2[2], 50)

	fmt.Println(*mp2[2]) // 输出: [30 40]
	fmt.Println(*mp2[3])

	// 打印 mp2[2] 和 mp2[3] 指针所指向的地址
	fmt.Printf("Address of mp2[2]: %p\n", mp2[2])
	fmt.Printf("Address of mp2[3]: %p\n", mp2[3])
}

```

结果:

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312061749546.png)​

‍

指针所指向的地址不相同的情况：

```go
package main

import "fmt"

func main() {
	// 定义一个 map，每个 key 对应一个整数切片
	mp1 := make(map[int][]int)

	// 添加多个值到同一个 key 对应的切片中
	mp1[1] = append(mp1[1], 10)
	mp1[1] = append(mp1[1], 20)

	fmt.Println(mp1) // 输出: map[1:[10 20]]

	// 定义一个 map，每个 key 对应一个整数切片的指针
	mp2 := make(map[int]*[]int)

	// 添加多个值到同一个 key 对应的切片中
	slice1 := []int{30, 40}
	mp2[2] = &slice1

	// 这里是一个新的实例，在内存中独立存在，与之前的slice1不同
	slice2 := []int{30, 40}
	mp2[3] = &slice2

	fmt.Println(*mp2[2]) // 输出: [30 40]
	fmt.Println(*mp2[3])
	// 打印 mp2[2] 和 mp2[3] 指针所指向的地址
	fmt.Printf("Address of mp2[2]: %p\n", mp2[2])
	fmt.Printf("Address of mp2[3]: %p\n", mp2[3])
}

```

结果：

​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202312061749191.png)​

# 总结

浅浅地理解map与slice的差异：

对于slice切片，它的核心强调的是容量，能装纳多少数据，但是map强调的是键值对的存储与索引

map底层更加复杂，在扩容上来讲，map扩容的效率要低，slice->数组；map->桶；map扩容操作时需要对所有键值对重新计算哈希值和桶的位置。这种操作需要花费更长的时间。

更深层次的理解见底层篇——《核心底层》

总结起来主要了以下几点：

* 什么是map，map格式与数据类型限制【尤其是key】，map特征，如何创建以及初始化map
* map的常规操作：访问，赋值，遍历，删除，判断key是否存在，获取长度等
* map的嵌套

其实还涉及很多问题，比如map的并发安全问题，syc.map相关，后续想到再巩固
