---
title: Go Interface详解(一)
date: 2019-09-29 23:28:51
tags: “golang"
categories: "golang"
---

# Go Interface 详解(一)

大家知道在Go语言中goroutine和channel是处理并发的两大基石,而接口同样也是Go语言中对于数据类型处理的基石

在Go语言编程中,几乎所有的数据结构都是围绕接口展开,数据结构的核心便是接口.其中同样精彩的还有它让Go实现面向对象或者是内容组织都变得非常的方便

## interface是什么

**interface是一种类型**

```go
{
    type Interface interface{
        Answer() //这是一个方法
    }
}
```

我们可以从它的定义可以看出使用了`type`关键字

对于interface的理解每个人都不一样,按我自己的话来说就是

*interface是一种具有一组方法的类型*

简单来说,就是interface是一组method签名的组合,我们可以通过interface来定义对象的一组行为

interface有两种类型

- `empty interface` 即Go允许不带任何方法的interface
- `non empty interface` 即附带了方法的interface

**如果一个类型实现了一个interface中所有的方法,我们便可以说该类型实现了其interface**

> 注意是实现了接口中所有的方法的类型才能说实现了该接口

- 因此我们可以发现,所有类型都实现了`empty interface`,因为任何一种类型至少实现了0个方法
- Go中没有显式的关键字用来实现interface,你只需要实现该接口中所有方法即可,Go会自动检查,而不像java中的接口,若你要实现该接口,必须显示声明出来.

## interface的值

既然interface是一种类型,那么interface变量存储的值是什么呢

**interface 变量存储的是实现者的值**

即这个变量里面可以存实现这个interface的任意类型的对象

下面我们来看一个简单的例子

```go
package main

import "fmt"

/**
 * user: ZY
 * Date: 2019/9/28 14:50
 */


type Answer interface {
	Set(int)
	Get()int
}

type  Me struct{
	Value int
}

func (m *Me)Set(value int){
	m.Value=value
}

func (m Me)Get()int{
	return m.Value
}

func Result(answer Answer){
	answer.Set(1)
	fmt.Println(answer.Get())
}

func main(){
	me:=Me{}
	Result(&me)
}
//out
//1
```

我们可以很清晰的看到`struct Me`实现了`interface Answer`的所有方法,在声明函数`Result`中直接传入interface变量,使用该接口中的所有方法,而在将`struct Me`即作为interface变量传入函数中则完成了一次interface类型的使用

这也体现出**如果有多种类型实现了某个interface,这些类型都可以作为interface的变量,其值也可以通过interface变量存储**

```go
func main(){
	me:=Me{1}
	var a Answer
	a=&me
	a.Set(2)
	fmt.Println(a.Get())
}
//out
//2
```

从上面代码可以看出, interface 的变量中存储的是实现了 interface 的类型的对象值，这种能力是 [duck typing](http://en.wikipedia.org/wiki/Duck_typing)。

> 在使用 interface 时不需要显式在 struct 上声明要实现哪个 interface ，只需要实现对应 interface 中的方法即可，Go 会自动进行 interface 的检查，并在运行时执行从其他类型到 interface 的自动转换，即使实现了多个 interface，go 也会在使用对应 interface 时实现自动转换，这就是 interface 的魔力所在。

## empty interface 

> empty interface:没有方法的interface

根据定义可以知道，所有的类型都实现了`interface{}`

如果某参数是`interface{}`类型，这个参数就可以传入任何类型变量

```go
func Demo(i interface{}){
}
```

虽然看起来函数参数用`interface{}`非常诱惑，但是其中的陷阱也非常多

函数被调用时在函数内部i并不是表示任何类型，虽然函数参数可以接受任何类型，并不代表i就是任何类型,在函数`Demo`中i仅仅是一个interface类型

之所以函数可以接受任何类型是因为Go在执行时传到函数的任何类型都被自动转化为`interface{}`

> 如果想要了解这其中的过程是如何进行转换的，还有i存储的值是怎样做到可以接受任何类型
>
> 大家可以去看看 [Russ Cox 关于 interface 的实现](https://research.swtch.com/interfaces) 

既然空的interface可以接受任何类型的参数

那么我们想到可不可以用`interface{}`类型的切片即`slice`是不是也可以接受任何类型的`slice`呢

接下来我们来看一个例子

```go
func Print(value []interface{}) { //1
	for _, v := range value {
		fmt.Println(v)
	}
}

func main(){
	s := []string{"good", "posture"}
	printAll(s)
}

//error
//cannot use s (type []string) as type []interface {} in argument to Print
```

看到这里，我们也会很疑惑为什么这里会报错呢

首先，我们之前说到为什么`interface{}`可以接受任何类型的参数，是因为Go会将传入的类型自动转换成`interface{}`，所以我们可以看出

- Go不会对`slice`进行转换，也就不会类型`interface{}`的`slice`进行转换

至于为什么不会进行自动转换呢

是关于`interface{}`内部的存储模式，简单来说，也就是它的内部占用了两个字长的存储空间

- 一个是自身存储的方法数据
- 一个是指向其存储值的指针

所以在`[]interface{}`其占用空间长度就为`N*2`，但是普通`slice`的长度为`N`,两种`slice`实际存储的大小是由区别的

但是还是有解决办法的

那就是手动进行自动转换，即在声明`[]interface`时通过`make`先申请与普通`slice`相同的长度，然后再进行赋值，代码我这里就不贴了

## interface变量存储类型

我们知道`interface`变量被多种类型实现的时候，我们怎么才能反向知道这个变量实际时保存了哪一个类型的对象呢？

下面介绍两种常用的方法

- Go独有的`Comma-ok`断言

让我们先来看看如何使用

> `Comma-ok`可以直接判断是否是该类型的变量：
>
> value, ok = element.(T)
>
> 这里value就是变量的值，ok是一个bool类型，element是interface变量，T是断言的类型。
>
> 如果element里面确实存储了T类型的数值，那么ok返回true，否则返回false。

```json
type Element interface {
}

type Elements []Element

type Answer struct{
	result string
}

//实现了fmt.Stringer
func (a Answer)String()string{
	return "result:"+a.result
}

func main(){
	elements:=make(Elements,3)
	elements[0]= "1"
	elements[1]= 2
	elements[2]=Answer{"result"}
	for k,v:=range elements{
		if value,ok:=v.(int);ok{
			fmt.Printf("%d : %d\n",k,value)
		}else if value,ok:=v.(string);ok{
			fmt.Printf("%d : %s\n",k,value)
		}else if value,ok:=v.(Answer);ok{
			fmt.Printf("%d : %s\n",k,value)
		}
	}
}

//out
//0 : 1
//1 : 2
//2 : result:result
```

- Switch测试

下面我们直接来看代码

```go
//
//变量定义与上面相同
//

func main(){
	elements:=make(Elements,3)
	elements[0]= "1"
	elements[1]= 2
	elements[2]=Answer{"result"}
	for k,v:=range elements{
		switch value:=v.(type) {
		case int:
			fmt.Printf("%d : %d\n",k,value)
		case string:
			fmt.Printf("%d : %s\n",k,value)
		case Answer:
			fmt.Printf("%d : %s\n",k,value)
		}
	}
}

//out
//0 : 1
//1 : 2
//2 : result:result
```

> 这种区分能力也被叫做[Type assertions](https://tour.golang.org/methods/15) (类型断言)
>
> 但是一定要注意使用的地方，并不是所有地方都可以使用类似`element.(type)`

## interface变量的receiver选择

我们在前面有一个例子中用到

`Result(&me)`

即是用的变量`me`的指针类型，为什么我们没有选择直接使用变量呢

如果使用编译器则会报错

```out
cannot use me (type Me) as type Answer in argument to Result:
	Me does not implement Answer (Set method has pointer receiver)
```

我们应该可以看出问题，那就是其中Me中的Set方法的receiver时一个指针类型也就是pointer *Me*

虽然在使用interface时并没有显示说出方法receiver是`value receiver`或者`pointer receiver`

上面Me定义的两个方法一个是`value receiver`，一个是`pointer receiver`，所以在interface在进行声明时还是无法同时满足

即可以看出

**Go中函数都是按值传递即passed by value**

那么现在我们将Set方法变为`value receiver`，同时也将变量取指针和普通变量

```go
type Answer interface {
	Set(int)
	Get()int
}

type  Me struct{
	Value int
}

func (m Me)Set(value int){
	m.Value=value
}

func (m Me)Get()int{
	return m.Value
}

func Result(answer Answer){
	answer.Set(1)
	fmt.Println(answer.Get())
}

func main(){
	me:=Me{}
	Result(&me)
	Result(me)
}

//out
//0
//0

```

我们看到执行结果，可以明白无论是`pointer`还是`value`都可以正确地执行

但是我们也可以看到执行结果中在调用函数Result时都没有改变其Value的值即函数Set并没有改变真正的值

导致这一现象的原因，也就是**Go的自动转换**

简单来说

**Go会把`pointer`隐式转换成`value`，但反过来也不行**

这也就是C语言中的函数的按值传递，对指针有了解的小伙伴就一定知道这种情况

## interface嵌入

Go语言中还有具有魅力的地方

那就是内置的逻辑语法，即将相同的逻辑引入到interface里面

简单来说就是

如果一个interface1作为interface2的一个嵌入字段，那么interface2就隐式包含了interface1里面的method

我们可以在源码中许多包里看到这种用法

- 源码包container/heap里面有这样一个定义

  ```go
  type Interface interface {
  	sort.Interface //嵌入字段sort.Interface
  	Push(x interface{}) //a Push method to push elements into the heap
  	Pop() interface{} //a Pop elements that pops elements from the heap
  }
  
  ```

  我们可以看到其中潜入字段`sort.Interface`，而`sort.Interface`里面包含以下三个方法，也就隐式地包含进来

  ```go
  type Interface interface {
  	// Len is the number of elements in the collection.
  	Len() int
  	// Less returns whether the element with index i should sort
  	// before the element with index j.
  	Less(i, j int) bool
  	// Swap swaps the elements with indexes i and j.
  	Swap(i, j int)
  }
  
  ```

- 源码包io.ReadWriter,它包含了io包下面的Reader和Writer两个interface

  ```go
  // io.ReadWriter
  type ReadWriter interface {
  	Reader
  	Writer
  }
  
  ```

  

## 参考

<https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/>

<https://juejin.im/post/5a6873fd518825734501b3c5#heading-6>

<https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.6.md>