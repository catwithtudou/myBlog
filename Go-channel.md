---
title: Go channel
date: 2019-09-30 00:51:54
tags: "golang"
categories: "golang"
---

# Go channel

Go语言与其他语言最具有区分的地方，也被称作此语言中最好的设计

那就是Golang中的**goroutine和channel**

当你了解通过Go来实现并发是多么的容易，并且并发性能优秀，你肯定会爱上这一个语言

## channel

channel相当于是一个通信机制也就是管道的概念，可以在程序任何地方进行通信，其本质也是一个FIFO的队列结构

但是它不仅仅是进行通信或者说是数据共享，它还可以利用自身的阻塞机制来实现超时，锁，定时等操作

最主要的是它线程是安全的，并不需要额外加锁

> channel的魅力只有当你真正运用它的时候，你才会发现它的简洁与优雅，相比于其他语言实现相同的功能你便可以发现其中的差异

### Brief introduction

它的使用方法非常的简单

- 操作符（非常形象表达出管道概念）

  `<-` 

   `->`

- make进行创建

```go
ch <- v    // 发送值v到Channel ch中

v := <-ch  // 从Channel ch中接收数据，并将数据赋值给v

ch := make(chan int) //make创建出一个无缓存的channel
```

### How to defined type

Channel的类型有三种

- 只读
- 只写
- 可读可写

定义格式如下：

```go
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```

其中操作符便代表相应的方向，可以说是非常简单形象

```go
var a chan T          // 可以接收和发送类型为 T 的数据
var b chan<- string  // 只可以用来发送 string 类型的数据
var c <-chan int      // 只可以用来接收 int 类型的数据
```

其中当我们声明channel,必须使用`make`初始化该channel,还能设置容量

```go
make(chan int,10)
make(chan int)
```

容量大小表示Channel容纳的最大元素数量，即代表的是Channel的缓存的大小。

若没有设置容量或者为0，说明Channel没有缓存

这时就会产生会有两种类型

- 设置了缓存即为缓存通道
- 没有设置缓存即为非缓存通道

至于两者区别我们在下面再说

### How to use

首先我们知道Channel本质是一个FIFO的队列，即接收和发送的数据的顺序是一致的

channel的receive支持`multi-valued assignment`，比如

```go
v,ok:=<-ch //检查Channel是否被关闭
```

#### Send

当我们需要往通道中发送数据时

只需要类似`ch <- 1`

定义如下：

```go
SendStmt = Channel "<-" Expression .
Channel  = Expression .
```

在通讯(communication)开始前channel和expression必选先求值出来(evaluated)，即expression必须是一个确定的值

而send被执行前通讯一直被阻塞着。

- 无缓存的channel只有在receiver准备好后send才被执行。如果有缓存，并且缓存未满，则send会被执行。

往一个已经被close

或者在无缓存且receiver没有准备好的情况下

又或者在有缓存并且在缓存空间已满情况下

向channel中继续发送数据会导致`panic`

#### Receive

当我们需要从通道中取出数据时

只需要类似`a:=<-ch`

同时也可以用`v,ok:=<-ch`来判断通道取出的是否为零值

在我们使用`<-ch`时程序会一直被阻塞，直到有数据可以接收

同样从一个nil channel中接收数据会一直被阻塞

但是从一个已经close的channel中接收数据不会被堵塞，而是立即返回未取出的数据又或者当channel中并无数据了也会返回元素类型的初始值即元值

#### Range&Close

`for value:=range ch`操作可以说是非常熟悉了，当我们遍历slice或者map时也是同样的用法

`close`也就是内置函数关闭channel的操作

让我们来看下面的例子

```go
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 1, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x + y //交换变量的便利写法
	}
	close(c) //关闭通道
}

func main() {
	c := make(chan int, 10)
    go fibonacci(cap(c), c) //cap(c)即为c的容量大小
	for i := range c { //不断读取channel数据，直到channel被显式关闭
		fmt.Println(i)
	}
}
```

> 其中记住应该在生产者的地方关闭channel，而不是消费的地方去关闭它，这样容易引起panic
>
> 另外channel不需要经常去关闭，只有当你确实没有任何发送数据或者想显式结束range循环等情况

#### Select

上面我们所看到的都是一个channel的情况，当我们遇到多个channel的时候应该如何操作呢

这时候`Select`关键词便可以出来大显身手了，它使我们可以轻松地监听或者是观察channel上的数据流动，类似于`switch`，但它只是用来处理通信操作

它的`case`可以是`send`,`receive`语句或者`default`

而`receive`语句可以将值赋值给一个或者两个变量。它必须是一个receive操作。

让我们来看官方给出的例子

```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:  //接收到数据后返回
			fmt.Println("quit")
			return
		}
	}
}
func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0 //当协程执行完上述循环后向quit中发送数据
	}()
	fibonacci(c, quit)
}
```

看到这里，我们会疑惑，万一同时有多个case需要处理，那怎么办？

Go处理这种情况时会伪随机的选择一个case进行处理

如果没有case需要处理，则会选择`default`去处理，若没有`default`语句时则`select`语句会被阻塞（`select`默认为阻塞），直到某个case需要处理

`select`语句和`switch`语句一样，它不是循环，它只会选择一个case来处理，如果想一直处理channel，你可以在外面加一个无限的for循环，即向上述例子中写的一样

#### timeout

channel的阻塞可以很好地利用来处理超时的情况或者说定时器的情况

比如有时候会出现goroutine阻塞的情况，若我们想要避免整个程序进入阻塞情况，我们就可以利用我们刚刚学到的`select`来设置超时的case

让我们来看看下面的例子

```go
func main() {
	c := make(chan int)
	o := make(chan bool)
	go func() {
		for {
			select {
				case v := <- c:
					println(v)
				case <- time.After(5 * time.Second):
					println("timeout")
					o <- true
					break
			}
		}
	}()
	<- o //若没有向o发送数据，主程序便会一直阻塞
    fmt.Println("the main has done")
}
```

> 我们可以在Go中自带的Timer和Ticker看到它正是利用channel这一特性来实现定时器的操作，其内部就是channel
>
> 让我们来实现一个简单的定时器
>
> ```go
> timer1 := time.NewTimer(time.Second * 2)
> <-timer1.C //2秒之前阻塞状态
> fmt.Println("Timer 1 expired")
> ```
>
> ```go
> ticker := time.NewTicker(time.Millisecond * 500)
> go func() {
> 	for t := range ticker.C { //每隔500ms向通道中发送数据
> 		fmt.Println("Tick at", t)
> 	}
> }()
> ```
>
> 其中Timer和Ticker的区别我这里就不再叙述了，有兴趣的小伙伴可以自己Google一下



### The features of Send and Receive

- **对于同一个通道，发送操作和接收操作之间都各自互斥**

  简单来说，也就是在同一时刻对于同一个通道

  Go程序运行只会执行其任意个发送操作中的某一个，直到这个元素被完全复制进该通道之后，其他发送操作才会执行，这也保证了数据的完整性和线程的安全性。

  针对接收操作也是同样的做法，这也就是互斥的原因。

  > 传入通道中的值会被复制也就是说传入的只是该变量值的副本

- **发送和接收操作中对元素值的处理都是不可分割的。用我的方式理解也就是类似数据库中的事务性，其结果只有复制完成和复制未完成，不会出现复制一部分的情况，即未完成之前都是阻塞的**

### The time of blocking

在上面我其实也已经提到了发送和接收操作什么时候会被阻塞的情况，我这里再总结一下

- 缓存通道
  - 如果通道已满，所有的发送操作就会阻塞，直到通道中有元素被取走
  - 如果通道已空，所有的接收操作就会阻塞，直到通道中有新的元素
- 非缓存通道
  - 无论发送操作还是接受操作一开始就是阻塞的，只有配对的操作出现才会开始执行。

> 日常开发中一般使用的是缓存通道，可以尽量避免阻塞，提高应用的性能

下面来看一个例子

```go
func sum(s []int,c chan int){
    sum:=0
    for _,v:=range s{
        sum+= v
    }
    c <- sum  //send sum to c
}

func main() {
	s := []int{7, 2, 8,1,0,-1,-2}
	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c
	fmt.Println(x, y)
}
```

从官方的例子上我们可以看到，`x,y:=<-c,<-c`这句会一直等待将值发送到channel中，在这过程中主程序是阻塞状态

> 其中我们了解到channel阻塞的这一特性后可以知道，当channel使用不当的时候便会出现死锁的情况，大家一定要注意，慎重的使用

## 参考

<https://colobu.com/2016/04/14/Golang-Channels/>

<https://juejin.im/post/5cd124cfe51d453a4d530d72#heading-1>

<https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.7.md>