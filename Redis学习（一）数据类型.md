---
title: Redis学习（一）数据类型
date: 2019-10-06 21:50:54
tags: ["redis","NoSQL"]
categories: "redis"
---

# Redis学习（一）数据类型

我们知道Redis的数据结构有五种，其分别是

- string
- list
- hash
- set
- zset

了解其Redis的数据类型可以帮助我们合理有效地去使用Redis

## what is the structure

### string

首先我们从最简单的`string`也就是字符串说起

`string`表示的是一个可变的字节数组

在redis里面字符串是**动态字符串即可以修改的字符串**

- 内部采用的是预分配冗余空间的方式来减少内存的频繁分配

简单来说就是当前字符串实际分配的空间容量`capacity`一般要高于其实际字符串长度`len`，而扩容的规则分为两种情况

- 当字符串长度小于1M时

  扩容是加倍现有的空间

- 当字符串长度超过1M时

  扩容时一次只会多扩1M的空间

> 在redis中字符串的最大长度为512M

### list

我们知道Redis中的`list`（列表）就是类似于我们熟知的`array`

不同的是`list`的存储结构用的是链表而不是数组，而且里面的链表还是双向链表，且list列表允许重复元素

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191001175207.png)

说到链表我们就可以从链表这一数据结构的优劣势来发现`list`什么地方使用起来比较合适

- 其优势在于定点插入删除元素，尤其是首尾部分，并且它并不指定大小扩展起来比较方便
- 其劣势在于其查找数据效率较低，因为不具有随机访问性

> 如果再深入一些，你会发现
>
> Redis底层存储的还不是一个简单的linkedlist，而是被称作为快速链表quicklist的一个结构
>
> 在了解quicklist之前我们需要来了解一个ziplist即压缩列表的概念
>
> - 简单来说就是在列表元素使用一块连续的内存存储
>
> 我们知道普通的链表元素内存地址都是随机的，是通过指针来连接，如果链表需要的附加指针空间太大就会比较浪费空间
>
> Redis就是采用将链表和ziplist结合起来组成了quicklist，即将多个ziplist使用双向指针串起来使用。
>
> 这样使用的话既满足了链表在插入删除上的性能，又减少了空间的浪费

### hash

熟悉Java的同学肯定知道HashMap这一容器，Redis中的hash即哈希就与HashMap十分类似几乎没有什么区别

说到哈希就是我们熟知的key-value模式

在Redis的hash又有一些不同的地方，它采用的是二维结构（你可以想象成二维数组）

- 第一维为数组

  存储链表的头指针

- 第二维为链表

  存储hash中的key-value

我们知道Redis大体上就是类似于key-value模式，其key也就是其键，而value就取决不同的数据结构

所以通过键即kye查找元素时，首先计算key的hashcode，然后用hashcode对于存储的`hash`结构数组的长度进行取模定位相应位置，我们上面说到第一维存储的就是链表的头指针即表头，然后从表头往下遍历找到相应元素的value值

> 哈希的第一维数组的长度也是2^n

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191001173522.png)

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191001173539.png)

### set

Redis中的set也叫做集合，类似于Java中的HashSet且其内部是通过HashMap实现的

而set内部也是使用的hash结构，是所有的value都指向同一个key即都指向同一个内部值

其实这样看下来它其实类似于list类型，它可以认为是没有排序的字符串集合，但是它与list不同的是不允许出现相同的元素，如果多次添加相同元素，set仅保留该元素的一份拷贝，且在查找效率上set因内部是hash所以会比list高很多

> 和list类型相比，set类型还有一个非常重要的特性，即它可以在服务器端完成多个集合之间的聚合计算操作
>
> 如：[`SUNION`](https://itbilu.com/database/redis/sunion)、[`SUNIONSTORE`](https://itbilu.com/database/redis/sunionstore)和[`SDIFFSTORE`](https://itbilu.com/database/redis/sdiffstore)。由于这些操作均在服务端完成，因此效率极高，而且也节省了大量的网络I/O开销。

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191001175017.png)

### sortedset

在Redis里面它被称作有序集合，它与集合（set）的类似，都是字符串的集合，且都不允许重复成员存在

而它们的主要区别在于有序集合中每一个成员都会一个分数（score）与之关联，而Redis通过可以这个分数来对集合成员进行有序排序

> score与元素的关系等价于Map<string,Map<string,Double>>
>
> score表示的更多的是元素的权重，而它是可重复的

sortedset底层使用了两个数据结构

- hash

  其作用就是集合中的key-value且保证value的唯一性

- 跳跃列表

  而它的作用便是给元素的value根据score排序，或者根据score范围获取元素列表

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191001180321.png)

> Redis 的有序集合类型操作效率非常高，是其它同类数据库所难以实现。在有序集合中添加、删除或更新一个元素的操作速度都非常快，其时间复杂度为集合中成员数量的对数。由于有序集合中的成员在集合中的位置是有序的，所以即使是操作集合中部的成员也会非常高效。

## 参考

<https://juejin.im/post/5b53ee7e5188251aaa2d2e16#heading-4>

<https://itbilu.com/database/redis/NJKsqw-9b.html>