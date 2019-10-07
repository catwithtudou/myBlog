---
title: Go goroutine
date: 2019-09-29 23:27:16
tags: "golang"
categories: "golang"
---

# Go goroutine

Go语言与其他语言最具有区分的地方，也被称作此语言中最好的设计

那就是Golang中的**goroutine和channel**

当你了解通过Go来实现并发是多么的容易，并且并发性能优秀，你肯定会爱上这一个语言

## goroutine

### 简介

goroutine的本质是通过协程来实现并行设计，而它的本质也是协程

goroutine的使用方式非常的简单，只需要`go`这一关键字便可启动这一个协程，并且它是处于异步方式运行，你并不需要等它运行完成以后在执行以后的代码

```go
go func() //通过go关键字来启动一个协程来运行相关函数
```

### 原理

在进行实现原理之前我们先复习一下相关概念

#### 相关概念

- **并发**

  一个cpu上能同时执行多项任务，在很短时间内，cpu来回切换任务执行(在某段很短时间内执行程序a，然后又迅速得切换到程序b去执行)，有时间上的重叠（宏观上是同时的，微观仍是顺序执行）,这样看起来多个任务像是同时执行，这就是并发。

- **并行**

  当系统有多个CPU时,每个CPU同一时刻都运行任务，互不抢占自己所在的CPU资源，同时进行，称为并行。

- **进程**

  cpu在切换程序的时候，如果不保存上一个程序的状态（也就是我们常说的context--上下文），直接切换下一个程序，就会丢失上一个程序的一系列状态，于是引入了进程这个概念，用以划分好程序运行时所需要的资源。因此进程就是一个程序运行时候的所需要的基本资源单位（也可以说是程序运行的一个实体）。

- **线程**

  cpu切换多个进程的时候，会花费不少的时间，因为切换进程需要切换到内核态，而每次调度需要内核态都需要读取用户态的数据，进程一旦多起来，cpu调度会消耗一大堆资源，因此引入了线程的概念，线程本身几乎不占有资源，他们共享进程里的资源，内核调度起来不会那么像进程切换那么耗费资源。

- **协程**

  协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。因此，协程能保留上一次调用时的状态（即所有局部状态的一个特定组合），每次过程重入时，就相当于进入上一次调用的状态，换种说法：进入上一次离开时所处逻辑流的位置。线程和进程的操作是由程序触发系统接口，最后的执行者是系统；协程的操作执行者则是用户自身程序，goroutine也是协程。

#### 调度模型简介

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20190929204732.png)

上面的图片便是让goroutine拥有强大的并发能力的调度模型，即被称为GPM调度模型

Go的调度器内部有四个重要结构的:M,P,S,Sched（未给出）

- M:M代表内核级线程，一个M就是一个线程，goroutine就是跑在M之上的；M是一个很大的结构，里面维护小对象内存cache（mcache）、当前执行的goroutine、随机数发生器等等非常多的信息
- G:代表一个goroutine，它有自己的栈，instruction pointer和其他信息（正在等待的channel等等），用于调度。
- P:P全称是Processor，处理器，它的主要用途就是用来执行goroutine的，所以它也维护了一个goroutine队列，里面存储了所有需要它来执行的goroutine
- Sched：代表调度器，它维护有存储M和G的队列以及调度器的一些状态信息等。

关于调度的具体实现我这里就不贴出了，有兴趣的小伙伴可以去看看 [go语言之行--golang核武器goroutine调度原理、channel详解](https://www.cnblogs.com/wdliu/p/9272220.html)

### 使用

#### 基础使用

> 设置goroutine运行的CPU数量，最新版本的go已经默认已经设置了。
>
> ```go
> num := runtime.NumCPU()    //获取主机的逻辑CPU个数
> runtime.GOMAXPROCS(num)    //设置可同时执行的最大CPU数
> ```

让我们直接来看一个例子

```go
func say(s string) {
	for i := 0; i < 5; i++ {
		fmt.Println(s)
	}
}

func main() {
	go say("world") //开一个新的Goroutines执行
	say("hello") //当前Goroutines执行
	time.Sleep(2 * time.Second)
}
```

其返回结果

```go
hello
hello
hello
world
world
world
world
world
hello
hello
```

我们可以看到`go`关键字很方便的就实现了并发编程。

 上面的一个goroutine运行在同一个进程里面，共享内存数据。

> 设计上我们要遵循：不要通过共享来通信，而要通过通信来共享。

#### 异常捕捉

当启动多个goroutine时，如果其中一个goroutine异常了，并且我们并没有对进行异常处理，那么整个程序都会终止出现`panic`,而导致其他进程失效

所以我们最好每个goroutine所运行的函数都做异常处理，异常处理采用`recover`关键字

让我们再来一个例子说明

```go
func say(a []int,s int) {
	defer func(){
	   err:=recover()
	   if err!=nil{
	   	 log.Println("error")
	   }
	}()
	a[s]=s
	fmt.Println(s)
}

func main() {
	arrya:=make([]int,4)
	for i:=0;i<5;i++{
		go say(arrya,i)
	}
	time.Sleep(2 * time.Second)
}
```

其返回结果

```go
0
1
2
3
2019/09/29 21:11:56 error
```

#### 同步

我们在使用goroutine经常会出现的一个问题是

当主程序退出时还有goroutine并没有执行完，而此时goroutine也会退出

出现这种情况，是因为goroutine是异步执行的

如果想要实现同步的情况即等待所有goroutine任务执行完毕才退出的话，一般的解决方法是通过以下两种

##### 使用channel

我们知道channel在发送或者接受的时候是堵塞的，且channel在所有协程中通信，我们可以利用这一点去实现同步

```go
var d int

func say(done chan int) {
	fmt.Println(d)
	d++
	if d==5{
		done<- d
	}

}

func main() {
	tasks := make(chan int, 5)
	for i := 0; i < 5; i++ {
		go say(tasks)//开启5个协程去运行函数
	}
	
	r:=<-tasks //若通道无输入则会堵塞
	fmt.Printf("%d :done\n",r)

	close(tasks)
}
```

其返回结果

```go
0
0
0
0
0
5 :done
```

##### 使用sync

sync包中有一个`WaitGroup`可用来进行队列堵塞，主程序通过Wait堵塞，直到等待队列为0

```go
func say(i int,wait *sync.WaitGroup){
	fmt.Println(i)
	defer wait.Done() //每次函数运行完成即协程运行完成，waitGroup队列数减1
}

func main() {
	var waitGroup sync.WaitGroup

	for i:=0;i<10;i++{
		waitGroup.Add(1) //在启动goroutine前，waitGroup队列加1
		go say(i,&waitGroup)
	}
	waitGroup.Wait() //等待队列数为空，主程序结束
}

```

#### 协程通讯

前面我们就已经提到goroutine可以通过channel进行通信或者数据共享，全局变量也可以进行数据共享

goroutine本质是协程，可以说不受内核调度，而只受Go中调度器管理的线程

下面我们通过一个例子来理解,即一个简单的广播

```go
func Boss(word string,num int,channel chan string,wait *sync.WaitGroup){
	for i:=0;i<num;i++ {
		channel <- word
	}
	defer func() {
		fmt.Println("Boss has done")
		wait.Done()
	}()
}

var done int = 0

func Worker(channel chan string,wait *sync.WaitGroup){
	word:=<-channel
	done++
	fmt.Printf("%d recive %s\n",done,word)
	defer func() {
		wait.Done()
	}()
}

func main(){
	channel:=make(chan string,10)
	var waitGroup sync.WaitGroup

	go Boss("Work",10,channel,&waitGroup)
	waitGroup.Add(1)

	for i:=0;i<10;i++{
		go Worker(channel,&waitGroup)
		waitGroup.Add(1)
	}
	waitGroup.Wait()
}

```

其返回结果

```go
2 recive Work
10 recive Work
3 recive Work
4 recive Work
5 recive Work
6 recive Work
7 recive Work
8 recive Work
9 recive Work
Boss has done
1 recive Work

```

## 参考

<https://www.cnblogs.com/wdliu/p/9272220.html>

