---
title: 微服务下分库分表的思考
date: '2023-10-16 14:22:08'
updated: '2023-10-16 17:45:20'
excerpt: 比较成体系关于分库分表的思考，持续更新....
tags:
  - mysql
categories:
  - MySQL
permalink: /post/my-thinking-about-the-mysql-database-table-zmjl3t.html
comments: true
toc: true
---

# 微服务下分库分表的思考

> 分库分表的这个名词再常见不过了，一开始的理解不够，随着对mysql理解的加深。
>
> 关于这个问题目前我的整体前置思路是：Mysql的瓶颈在哪里——InnoDB存储引擎——MySQL单表存储的瓶颈以及瓶颈推演与测试——分库分表，内容不一定绝对，但是思路是完整、有迹可循的。
>
> 关联文章：Mysql单表存储数据量瓶颈推演（2000W左右）

**目录**

1. 什么是分库分表？
2. 为什么需要分库分表？
3. 如何分库分表？
4. 什么时候开始考虑分库分表？
5. 分库分表会导致哪些问题？如何解决？
6. 分库分表中间件？

PS: 这里有个小插曲，关于分区的问题，这一点我是在架构师考试备考（数据库）中遇到的问题，mysql为什么好像从来没有听说分区的相关内容，经查资料才了解到数据库按理说是支持分区的，所谓的是将一个表按照一定规则水平划分成多个子表，每个子表存储一部分数据。分区是针对单个表的，个人理解上是物理上可能跨磁盘、跨系统，但本质上还是一张逻辑表，主要的目的是提高查询效率和管理大型表的数据，减少索引长度和IO操作等问题。MySQL不支持分区，但是可以通过其他方式来实现分区的功能。比如，可以通过应用程序来实现分区的功能，或者使用其他数据库管理系统，比如Oracle、SQL Server等，它们都支持分区。MySQL不支持分区是由于历史原因、成本问题和性能问题所致。追根揭底还是技术栈范围扩大后，很多概念是宏观的，很难保证一致性。比如说只聊数据库，会聊关系模式、关系代数、-（前面这些都是关系数据库讲的，我只用Redis的话谈什么这些）数据库设计、聊数据库优化技术，这些东西太过宏观，像是“基础”可我本质上还是觉得完全就不是一类东西，只能算是时代的眼泪，曾经的思想参考。

---

# **1. 什么是分库分表**

**分库**：就是一个数据库分成多个数据库，部署到不同机器。单体架构下就没有分库分表，比较单一，SOA到微服务的发展中演进的

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858982.jpg)​

**分表**：就是一个数据库表分成多个表。

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858913.jpg)​

# 2.为什么需要分库分表呢？

如果业务量剧增，数据库可能会出现性能瓶颈，这时候我们就需要考虑拆分数据库。从这几方面来看：

* **磁盘存储（很容易想到，这边便硬件层面的因素）**

业务量剧增，MySQL单机磁盘容量会撑爆，拆成多个数据库，磁盘使用率大大降低。

* **并发连接支撑（这个是mysql本身的瓶颈）**

我们知道数据库连接是有限的。在高并发的场景下，大量请求访问数据库，MySQL单机是扛不住的！当前非常火的**微服务架构**出现，就是为了应对高并发。它把**订单、用户、商品**等不同模块，拆分成多个应用，并且把单个数据库也拆分成多个不同功能模块的数据库（**订单库、用户库、商品库**），以分担读写压力。

***为什么需要分表？***

数据量太大的话，SQL的查询就会变慢。如果一个查询SQL**没命中索引**，千百万数据量的表可能会拖垮这个数据库。

即使SQL命中了索引，如果表的数据量**超过一千万**的话，**查询也是会明显变慢的**。这是因为索引一般是B+树结构，数据千万级别的话，B+树的高度会增高，查询就变慢啦。

Mysql单表存储数据量瓶颈推演（2000W左右）

**MySQL的B+树的高度怎么计算的呢？** 

InnoDB存储引擎最小储存单元是页，一页大小就是16k。B+树叶子存的是数据，内部节点存的是键值+指针。索引组织表通过非叶子节点的二分查找法以及指针确定数据在哪个页中，进而再去数据页中找到需要的数据，B+树结构图如下：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858114.jpg)​

假设B+树的高度为2的话，即有一个根结点和若干个叶子结点。这棵B+树的存放总记录数为=根结点指针数*单个叶子节点记录行数。

* 如果一行记录的数据大小为1k，那么单个叶子节点可以存的记录数  `=16k/1k =16`​
* 非叶子节点内存放多少指针呢？我们假设主键ID为**bigint类型，长度为8字节**(**面试官问你int类型，一个int就是32位，4字节**)，而指针大小在InnoDB源码中设置为6字节，所以就是 `8+6=14 ​`​字节，`16k/14B =16*1024B/14B = 1170`​

因此，一棵高度为2的B+树，能存放`1170 * 16=18720`​条这样的数据记录。

同理一棵高度为`3`​的B+树，能存放`1170 *1170 *16 =21902400`​，大概可以存放两千万左右的记录。B+树高度一般为1-3层，如果B+到了4层，查询的时候会**多查磁盘**的次数，SQL就会变慢。

因此单表数据量超过千万->就需要考虑分表啦。

> 这个地方从两种角度分析：颗粒度不一样，层面也不一样
>
> 1.页的细节角度
>
> 16K的页内结构
>
> ​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858917.png)​
>
> ​![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858148.png)​
>
> 这种角度：
>
> * 非叶子节点内指向其他页的数量为 x
> * 叶子节点内能容纳的数据行数为 y
> * B+ 数的层数为 z
>
> 页的结构，索引也也不例外，都会有 File Header (38 byte)、Page Header (56 Byte)、Infimum + Supermum（26 byte）、File Trailer（8byte）, 再加上页目录，大概 1k 左右。
>
> 我们就当做它就是 1K, 那整个页的大小是 16K, 剩下 15k 用于存数据，在索引页中主要记录的是主键与页号，主键我们假设是 Bigint (8 byte), 而页号也是固定的（4Byte）, 那么索引页中的一条数据的大小=8+4=12byte。15K=15*1024B
>
> 所以 **非叶子节点内指向其他页的数量x ​**=15*1024/12≈1280 行。
>
> 叶子节点和非叶子节点的结构是一样的，同理，能放数据的空间也是 15k。
>
> 但是叶子节点中存放的是真正的行数据，这个影响的因素就会多很多，比如，字段的类型，字段的数量。每行数据占用空间越大，页中所放的行数量就会越少。
>
> 这边我们暂时按一条行数据 1k 来算，那一页就能存下 15 条，Y = 15*1024/1000 ≈15。
>
> 算到这边了，是不是心里已经有谱了啊。
>
> Total 总数据行数
>
> 根据上述的公式，Total =x^(z-1) *y，已知 x=1280，y=15：
>
> * 假设 B+ 树是两层，那就是 z = 2， Total = （1280 ^1 ）*15 = 19200
> * 假设 B+ 树是三层，那就是 z = 3， Total = （1280 ^2） *15 = 24576000 （约 2.45kw）
>
> ​![8b14ff08d4e0ace39a16f474f393a8f](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858130.jpg)​
>
> 2.磁盘块角度（上述即是）：
>
> ​![de4f110029fd2c2442e392b10bbe771](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858346.jpg)​

‍

# **3. 如何分库分表**

> 水平即行，垂直即列，一横一竖，横即拆行，竖即拆列

## 3.1 垂直拆分

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858396.jpg)​

### **3.1.1 垂直分库**

在业务发展初期，业务功能模块比较少，为了快速上线和迭代，往往采用单个数据库来保存数据。数据库架构如下：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858120.jpg)​

但是随着业务蒸蒸日上，系统功能逐渐完善。这时候，可以按照系统中的不同业务进行拆分，比如拆分成**用户库、订单库、积分库、商品库**，把它们部署在不同的数据库服务器，这就是**垂直分库**。

垂直分库，将原来一个单数据库的压力分担到不同的数据库，可以很好应对高并发场景。数据库垂直拆分后的架构如下：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858861.jpg)​

### **3.1.2 垂直分表**

如果一个单表包含了几十列甚至上百列，管理起来很混乱，每次都`select *`​的话，还占用IO资源。这时候，我们可以将一些**不常用的、数据较大或者长度较长的列**拆分到另外一张表。

比如一张用户表，它包含`user_id、user_name、mobile_no、age、email、nickname、address、user_desc`​，如果`email、address、user_desc`​等字段不常用，我们可以把它拆分到另外一张表，命名为用户详细信息表。这就是垂直分表：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858620.jpg)​

## 3.2 水平拆分

### **3.2.1 水平分库**

水平分库是指，将表的数据量切分到不同的数据库服务器上，每个服务器具有相同的库和表，只是表中的数据集合不一样。它可以有效的缓解单机单库的性能瓶颈和压力。

用户库的水平拆分架构如下：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858351.jpg)​

### **3.2.2 水平分表**

如果一个表的数据量太大，可以按照某种规则（如`hash取模、range`​），把数据切分到多张表去。

一张订单表，按`时间range`​拆分如下：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858379.jpg)​

### 3.3. 水平分库分表策略

分库分表策略一般有几种，使用与不同的场景：

* range范围
* hash取模
* range+hash取模混合

#### **3.3.1 range范围**

range，即范围策略划分表。比如我们可以将**表的主键**，按照从`0~1000万`​的划分为一个表，`1000~2000万`​划分到另外一个表，以此类推。如下图：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858200.jpg)​

<u>当然，有时候我们也可以按</u>​<u>**时间范围**</u><u>来划分，如不同年月的订单放到不同的表，它也是一种range的划分策略。</u>

**这种方案的优点：**

* 这种方案有利于扩容，不需要数据迁移。假设数据量增加到5千万，我们只需要水平增加一张表就好啦，之前`0~4000万`​的数据，不需要迁移。

**缺点：**

* 这种方案会有热点问题，因为订单id是一直在增大的，也就是说最近一段时间都是汇聚在一张表里面的。比如最近一个月的订单都在`1000万~2000`​万之间，<u>*平时用户一般都查最近一个月的订单比较多*</u>，请求都打到`order_1`​表啦，这就导致表的**数据热点**问题。

#### **3.3.2 hash取模**

hash取模策略：指定的路由key（一般是user_id、订单id作为key）对分表总数进行取模，把数据分散到各个表中。

比如原始订单表信息，我们把它分成4张分表：

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858963.jpg)​

* 比如id=1，对4取模，就会得到1，就把它放到第1张表，即`t_order_0`​;
* id=3，对4取模，就会得到3，就把它放到第3张表，即`t_order_2`​;

**这种方案的优点：**

* hash取模的方式，不会存在明显的热点集中问题。

**缺点：**

* 如果一开始按照hash取模分成4个表了，未来某个时候，表数据量又到瓶颈了，需要扩容，这就比较棘手了。比如你从4张表，又扩容成`8`​张表，那之前`id=5`​的数据是在（`5%4=1`​，即第一张表），现在应该放到（`5%8=5`​，即第`5`​张表），也就是说**历史数据要做迁移了**。

#### **3.3.3 range+hash取模混合**

既然range存在热点数据问题，hash取模扩容迁移数据比较困难，我们可以综合两种方案一起嘛，取之之长，弃之之短。

比较简单的做法就是，在拆分库的时候，我们可以先用**range范围**方案，比如订单id在0~4000万的区间，划分为订单库1，id在4000万~8000万的数据，划分到订单库2,将来要扩容时，id在8000万~1.2亿的数据，划分到订单库3。然后订单库内，再用**hash取模**的策略，把不同订单划分到不同的表。

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202311011858703.jpg)​

# **4. 什么时候考虑分库分表？**

前提：能不能切分就不要分，分了会极大的导致系统的复杂性。避免"过度设计"和"过早优化"。

在分库分表之前，不要为分而分，先尽力去做力所能及的事情，例如：升级硬件、升级网络、读写分离、索引优化等等。当数据量达到单表的瓶颈时候，再考虑分库分表。

4.1 什么时候分表？

如果你的系统处于快速发展时期，如果每天的订单流水都新增几十万，并且，订单表的查询效率明变慢时，就需要规划分库分表了。一般B+树索引高度是2~3层最佳，如果数据量千万级别，可能高度就变4层了，数据量就会明显变慢了。不过业界流传，一般500万数据就要**考虑分表**了，属于提前考虑，留好空间。

4.2 什么时候分库

这一点我的理解是看业务量，微服务架构下，微服务特别多，或者很多重要的业务要维护高并发高可用的要求，就要分库，比如说重点的业务单体抽取出来单独分库。

业务发展很快，还是多个服务共享一个单体数据库，数据库成为了性能瓶颈，就需要考虑分库了。比如订单、用户等，都可以抽取出来，新搞个应用（其实就是微服务思想），并且拆分数据库（订单库、用户库）。

综合来讲，考虑分库分表的无非以下方面：

① 数据量快速增长，当业务中数据量急速增长时

② 维护困难时：比如，备份时较为困难，**单表太大，备份时需要大量的磁盘IO和网络IO；对一个很大的表进行DDL修改时**，MySQL会锁住全表，这个时间会很长，这段时间业务不能访问此表，影响很大**。**如果使用pt-online-schema-change，使用过程中会创建触发器和影子表，也需要很长的时间。在此操作过程中，都算为风险时间。将数据表拆分，总量减少，有助于降低这个风险；**经常访问与更新**，就更有可能出现锁等待，将数据切分，用空间换时间，变相降低访问压力。

③ 业务需求：**需要对某些字段垂直拆分，本质上还是因用户增多导致的业务特点发生了变化**

举个例子，假如项目一开始设计的用户表如下：

```
id                   bigint             #用户的ID
name                 varchar            #用户的名字
last_login_time      datetime           #最近登录时间
personal_info        text               #私人信息
.....                                   #其他信息字段
```

在项目初始阶段，这种设计是满足简单的业务需求的，也方便快速迭代开发。而当**业务快速发展时，用户量从10w激增到10亿，用户非常的活跃**，每次登录会更新 last_login_name 字段，使得 user 表被不断update，压力很大。

而其他字段：id, name, personal_info 是不变的或很少更新的，此时在业务角度，就要将 last_login_time 拆分出去，新建一个 user_time 表。

personal_info 属性是更新和查询频率较低的，并且text字段占据了太多的空间。这时候，就要对此垂直拆分出 user_ext 表了

④ 安全性角度：鸡蛋不要放在一个篮子里。在业务层面上垂直切分，将不相关的业务的数据库分隔，因为每个业务的数据量、访问量都不同，不能因为一个业务把数据库搞挂而牵连到其他业务。利用水平切分，当一个数据库出现问题时，不会影响到100%的用户，每个库只承担业务的一部分数据，这样整体的可用性就能提高。

# **5. 分库分表会导致哪些问题**

**分库分表之后，也会存在一些问题：**

* 事务问题
* 跨库关联
* 排序问题
* 分页问题
* 分布式ID
* 数据迁移、扩容问题

5.1 事务问题

分库分表后，假设两个表在不同的数据库，那么本地事务已经无效啦，需要使用分布式事务了。

5.2 跨库关联

跨节点Join的问题：解决这一问题可以分两次查询实现；

冗余处理，反规范化设计：

1. **增加冗余列(复制某一列的数据) 这一点是真的好用**
2. 增加派生列(计算总和，平均值..)
3. 表合并(把部分来自不同表的常用列合并成新表)
4. 表分割(把数据拆分为常用和不常用，行拆分是比如订单信息，列拆分比如账户信息<额外的住址之类的并不常用，减少查询压力>) 题中要求是商品信息冗余，所以应该采用<增加冗余列>的方法

5.3 排序问题

跨节点的count,order by,group by以及聚合函数等问题：可以分别在各个节点上得到结果后在应用程序端进行合并。

5.4 分页问题

* 方案1：在个节点查到对应结果后，在代码端汇聚再分页。
* 方案2：把分页交给前端，前端传来pageSize和pageNo，在各个数据库节点都执行分页，然后汇聚总数量前端。这样缺点就是会造成空查，如果分页需要排序，也不好搞。

5.5 分布式ID

据库被切分后，不能再依赖数据库自身的主键生成机制啦，最简单可以考虑UUID，或者使用雪花算法生成分布式ID。

**UUID:**

UUID标准形式包含32个16进制数字，分为5段，形式为8-4-4-4-12的32个字符，例如：550e8400-e29b-41d4-a716-446655440000

UUID是主键是最简单的方案，本地生成，性能高，没有网络耗时。但缺点也很明显，由于UUID非常长，会占用大量的存储空间；另外，作为**主键建立索引和基于索引进行查询时都会存在性能问题**，在InnoDB下，UUID的无序性会引起数据位置频繁变动，导致分页。

**雪花算法：**

Twitter的snowflake算法解决了分布式系统生成全局ID的需求，生成64位的Long型数字，组成部分：

* 第一位未使用
* 接下来41位是毫秒级时间戳，41位的长度可以表示69年的时间
* 5位datacenterId，5位workerId。10位的长度最多支持部署1024个节点
* 最后12位是毫秒内的计数，12位的计数顺序号支持每个节点每毫秒产生4096个ID序列

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202310161750788.png)​

**好处：**毫秒数在高位，生成的ID整体上按时间趋势递增；不依赖第三方系统，稳定性和效率较高，理论上QPS约为409.6w/s（1000*2^12），并且整个分布式系统内不会产生ID碰撞；可根据自身业务灵活分配bit位。

**不足：**强依赖机器时钟，如果时钟回拨，则可能导致生成ID重复。

综上，结合数据库和snowflake的唯一ID方案，可以参考业界较为成熟的解法：**Leaf——美团点评分布式ID生成系统，并考虑到了高可用、容灾、分布式下时钟等问题。**

**数据迁移、扩容问题：**

当业务高速发展，面临性能和存储的瓶颈时，才会考虑分片设计，此时就不可避免的需要考虑历史数据迁移的问题。一般做法是先读出历史数据，然后按指定的分片规则再将数据写入到各个分片节点中。

此外还需要根据当前的数据量和QPS，以及业务发展的速度，进行容量规划，推算出大概需要多少分片（一般建议单个分片上的单表数据量不超过1000W）。

如果采用数值范围分片，只需要添加节点就可以进行扩容了，不需要对分片数据迁移。如果采用的是数值取模分片，则考虑后期的扩容问题就相对比较麻烦。

# **6. 分库分表中间件**

**目前流行的分库分表中间件比较多：**

* cobar
* Mycat
* Sharding-JDBC（当当）
* Atlas
* TDDL（淘宝）
* vitess（谷歌开发的数据库中间件）

​![图片](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/myblog/202310161746040.jpg)

​

> 参考资料：如何分库分表！

‍