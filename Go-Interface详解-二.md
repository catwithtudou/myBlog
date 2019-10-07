---
title: Go Interface详解(二)
date: 2019-09-29 23:29:34
tags: “golang"
categories: "golang"
---

# Go Interface 详解(二)

这一篇我们来更加深入地了解Go语言中interface的魅力

首先让我们来了解为什么要使用interface

或者说interface可以给我们带来什么

## Why Interface

在Gopher China 2017 大会上提出了许多有趣的东西

以下是 Gopher China 给出的三个理由

- **writing generic algorithm （泛型编程）**
- **hiding implementation detail （隐藏具体实现）**
- **providing interception points （提供拦截点）**

### writing generic algorithm

其实从严格意义来说

Golang并不支持泛型编程

我们知道在JAVA或者是C++等高级语言中使用泛型编程很简单,而Golang并不能与其他语言在此相比,这也成为了Golang诟病最多的一个地方

但是interface确实可以实现泛型编程

首先我们来看以标准库中的sort作为例子

```go
package sort

// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package.  The methods require that the
// elements of the collection be enumerated by an integer index.
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}

...

// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {
    // Switch to heapsort if depth of 2*ceil(lg(n+1)) is reached.
    n := data.Len()
    maxDepth := 0
    for i := n; i > 0; i >>= 1 {
        maxDepth++
    }
    maxDepth *= 2
    quickSort(data, 0, n, maxDepth)
}
```

我们可以看到`Sort`函数的形参就是一个`interface{}`,其中包含了三个方法即`Len()`,`Less(i,j int)`,`Swap(i,j int)`

我们在上篇文章了解到实现接口只需要把该接口所有方法都实现即可以为`interface{}`,

所以只要实现了三个方法,即可以使用`Sort`函数,这样就是实现了**泛型编程**

这里给出一个具体的例子

```go
type Number struct {
	num  int
}

func (p Number) String() string {
	return fmt.Sprintf("%d", p.num)
}

type ByNum []Number //自定义

func (a ByNum) Len() int           { return len(a) }
func (a ByNum) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByNum) Less(i, j int) bool { return a[i].num < a[j].num }

func main() {
	numbers := []Number{
		{1},
		{2},
		{4},
		{3},
	}

	fmt.Println(numbers)
	sort.Sort(ByNum(numbers))
	fmt.Println(numbers)
}


//out
//[1 2 4 3]
//[1 2 3 4]

```

另外 Fransesc 在 Gopher China 上还提到了一个比较有趣的东西和大家分享一下。在我们设计函数的时候，下面是一个比较好的准则。

> Be **conservative** in what you send, be **liberal** in what you accept. — Robustness Principle

对应到 Golang 就是：

> Return **concrete types**, receive **interfaces** as parameter. — Robustness Principle applied to Go

话说这么说，但是当我们翻阅 Golang 源码的时候，有些函数的返回值也是 interface。

### hiding implementation detail

隐藏具体的实现,其实也就是封装的概念

即接口内部实现方法的具体流程你是不知道的,你只能使用其接口中的方法来做相应的操作

在大会上Francesc 举了个 context 的例子。 context 最先由 google 提供，现在已经纳入了标准库，而且在原有 context 的基础上增加了：cancelCtx，timerCtx，valueCtx。

让我们直接来看一下 context 包的代码吧。

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    c := newCancelCtx(parent)
    propagateCancel(parent, &c)
    return &c, func() { c.cancel(true, Canceled) }
}
```

表明上 WithCancel 函数返回的还是一个 Context interface，但是这个 interface 的具体实现是 cancelCtx struct。

```go
// newCancelCtx returns an initialized cancelCtx.
func newCancelCtx(parent Context) cancelCtx {
    return cancelCtx{
        Context: parent,
        done:    make(chan struct{}),
    }
}

// A cancelCtx can be canceled. When canceled, it also cancels any children
// that implement canceler.
type cancelCtx struct {
    Context     //注意一下这个地方!!!!!!!!!!!!!!!!!!!!!!!!!!

    done chan struct{} // closed by the first cancel call.
    mu       sync.Mutex
    children map[canceler]struct{} // set to nil by the first cancel call
    err      error                 // set to non-nil by the first cancel call
}

func (c *cancelCtx) Done() <-chan struct{} {
    return c.done
}

func (c *cancelCtx) Err() error {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.err
}

func (c *cancelCtx) String() string {
    return fmt.Sprintf("%v.WithCancel", c.Context)
}
```

尽管内部实现上下面三个函数返回的具体 struct （都实现了 Context interface）不同，但是对于使用者来说是完全无感知的。

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)    //返回 cancelCtx
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc) //返回 timerCtx
func WithValue(parent Context, key, val interface{}) Context    //返回 valueCtx
```

### providing interception points

我想Francesc在这里想要表达的意思应该是装饰器或者动态调度,类似中间件的概念吧

他给出了一个例子

```go
type header struct {
    rt  http.RoundTripper
    v   map[string]string
}

func (h header) RoundTrip(r *http.Request) *http.Response {
    for k, v := range h.v {
        r.Header.Set(k,v)
    }
    return h.rt.RoundTrip(r)
}
```

> 这里补充一下Francesc还提到了一个非侵入式特性
>
> 这个解释可以通过对比Java中的interface实现需要显示的声明,即
>
> `public class MyWriter implements io.Writer {}`
>
> 这也意味着如果实现多个interface需要显示地写很多遍,同时package的依赖还需要进行管理
>
> 大家知道`Dependency is evil`
>
> 而Golang的写法则异常方便,比如实现io包中的Reader,Writer,ReadWriter接口
>
> ```go
> type Demo struct {}
> 
> func (io *Demo) Read(p []byte) (n int, err error) {...}
> func (io *Demo) Write(p []byte) (n int, err error) {...}
> 
> // io package
> type Reader interface {
>  Read(p []byte) (n int, err error)
> }
> 
> type Writer interface {
>  Write(p []byte) (n int, err error)
> }
> 
> type ReadWriter interface {
>  Reader
>  Writer
> }
> ```
>
> 这样便可以不用显示的import io package
>
> 为什么可以这样呢,是因为interface会在底层做动态的检测
>
> 但是任何事情都是有瑕疵的,它也会引入相关的问题
>
> 1. 性能下降。使用 interface 作为函数参数，runtime 的时候会动态的确定行为。而使用 struct 作为参数，编译期间就可以确定了。
> 2. 不知道 struct 实现哪些 interface。这个问题可以使用 guru 工具来解决。
>
> 综上,interface有好有坏,但可以肯定的是它使代码变得非常简洁

## interface 源码分析

任何事情都需要从源码了解其本质

让我们再来具体看一看interface具体源码的实现吧

### interface 结构

```go
    

type eface struct {
        _type *_type
        data  unsafe.Pointer
    }
    
type _type struct {
        size       uintptr // type size
        ptrdata    uintptr // size of memory prefix holding all pointers
        hash       uint32  // hash of type; avoids computation in hash tables
        tflag      tflag   // extra type information flags
        align      uint8   // alignment of variable with this type
        fieldalign uint8   // alignment of struct field with this type
        kind       uint8   // enumeration for C
        alg        *typeAlg  // algorithm table
        gcdata    *byte    // garbage collection data
        str       nameOff  // string form
        ptrToThis typeOff  // type for pointer to this type, may be zero
    }
    

```

我们都知道interface是可以存储method的,那么从源码中可以看到在底层的interface结构中,它是用两种`struct`来表示的

- iface

  iface 表示 non-empty interface 的底层实现。相比于 empty interface，non-empty 要包含一些 method。method 的具体实现存放在 itab.fun 变量里,我们可以从源码看到

  ```go
  type iface struct {
          tab  *itab
          data unsafe.Pointer
      }
      
  // layout of Itab known to compilers
  // allocated in non-garbage-collected memory
  // Needs to be in sync with
  // ../cmd/compile/internal/gc/reflect.go:/^func.dumptypestructs.
  type itab struct {
          inter  *interfacetype
          _type  *_type
          link   *itab
          bad    int32
          inhash int32      // has this itab been added to hash?
          fun    [1]uintptr // variable sized
      }
  ```

  我们从该iface中的itab中可以看到这个`struct`包含了一些关于interface本身的信息

  到这里可能会疑惑为什么一个fun变量可以包含这么多method

  其实原理上是来对原数据类型进行转换，即转换成<_type, unsafe.Pointer> 或者 <itab, unsafe.Pointer>

  这里对于 struct 满不满足 interface 的类型要求（也就是 struct 是否实现了 interface 的所有 method），是由编译器来检测的

  还有一个比较特殊的interfacetype，它就是我们定义interface时候的一种抽象表示

  比如 interface type 包含了 method A, B，则通过 fun 就可以找到这两个 method 的具体实现。

  ```go
  type interfacetype struct {
     typ     _type // 表示 concrete type
     pkgpath name
     mhdr    []imethod
  }
  
  type imethod struct {   //这里的 method 只是一种函数声明的抽象，比如  func Print() error
     name nameOff
     ityp typeOff
  }
  ```

- eface

  eface 表示 empty interface 的底层实现

## interface的内存结构

让我们再来看一下interface的内存结构

因为这是我们了解其interface效率的根本源头

我们在上一篇可以了解到interface实际由两个成员组成

- tab 指向虚表 （虚表描绘了实际的类型信息及该接口所需要的方法集）
- data 指向实际引用的数据

来看一个例子

```go
    type Stringer interface {
        String() string
    }
    
    type Binary uint64
    
    func (i Binary) String() string {
        return strconv.Uitob64(i.Get(), 2)
    }
    
    func (i Binary) Get() uint64 {
        return uint64(i)
    }
    
    func main() {
        b := Binary{}
        s := Stringer(b)
        fmt.Print(s.String())
    }
   
```

我们知道itable的结构体，是描述了type信息的一些元数据，然后是满足Stringger接口的函数指针列表

但是这里不是实际类型Binary的函数指针集

因此我们如果通过接口进行函数调用，实际也就是s.tab->fun()

这里也就是golang实现了虚表，它为每种类型创建了一个方法集，与C++不同的是接口的虚表是运行时专门生成的。

选择在运行时才生成虚表， 是因为虚表中的部分其实是不需要的，我们知道每一种接口类型和所有满足其接口的实体类型的组合就是其可能的虚表数量

例如，当例子中当首次遇见s := Stringer(b)这样的语句时，golang会生成Stringer接口对应于Binary类型的虚表，并将其缓存。

了解了这些，我们可以判断某些比如类型断言等情况的效率问题了

例如某类型有m个方法，某接口有n个方法，则很容易知道这种判定的时间复杂度为O(mXn)，不过可以使用预先排序的方式进行优化，实际的时间复杂度为O(m+n)。

## interface与nil

直接贴代码来帮助我们理解

```go
package main

import (
	"fmt"
	"reflect"
)

type State struct{}

func testnil1(a, b interface{}) bool {
	return a == b
}

func testnil2(a *State, b interface{}) bool {
	return a == b
}

func testnil3(a interface{}) bool {
	return a == nil
}

func testnil4(a *State) bool {
	return a == nil
}

func testnil5(a interface{}) bool {
	v := reflect.ValueOf(a)
	return !v.IsValid() || v.IsNil()
}

func main() {
	var a *State
	fmt.Println(testnil1(a, nil))
	fmt.Println(testnil2(a, nil))
	fmt.Println(testnil3(a))
	fmt.Println(testnil4(a))
	fmt.Println(testnil5(a))
}

//out
//false
//false
//false
//true
//true
```

> 我们在上面可以了解到一个interface{}类型的变量包含了2个指针，一个指针指向值的类型，另外一个指针指向实际的值 

对一个interface{}类型的nil变量来说，它的两个指针都是nil；但是var a *State传进去后，指向的类型的指针不为nil了，因为有类型了， 所以比较结果为为false。 

**interface 类型比较，必须要其中的两个指针都相等， 才能相等。**

## 参考

<https://juejin.im/post/5a6873fd518825734501b3c5#heading-3>

<http://legendtkl.com/2017/06/12/understanding-golang-interface/>