---
title: golang-interface解析
date: 2019-08-12 19:29:34
tags: "golang"
categories: "golang"
---

# golang-interface解析

> Go语言的主要设计者之一罗布·派克（ Rob Pike）曾经说过，如果只能选择一个Go语言的特 性移植到其他语言中，他会选择接口。可见接口在golang中的地位，及其对gloang这门语言所带来的活力。

## 什么么interface

我们现在可以设想一个场景

想象你要在路边打一辆出租车从A地到B地，你不需要知道出租车司机的国籍、性别，是人还是机器人，只要他能够把你从A地带到B地就可以。 

golang中的interface就是这个意思，你不要一个特定的类型，而是需要它实现你想要的功能。interface可以理解为你想要的功能集合。

> golang的设计理念中,一个类只需要实现了接口要求的所有函数，我们就说这个类实现了该接口

```go
//interface 定义你想要的功能
 type Driver interface {
    //能够把你从A地带到B地
    Drive(from Location, to Location) 
  }
//只要实现了你想要功能（Drive）的类型都满足你的要求
  func TakeRide(from Location, to Location, driver Driver){
    driver.Drive(from, to)
  }
//定义两个能够Drive的类型
  type Human struct {
  }
  func (p *Human) Drive(from Location, to Location){
    //implements the drive method
  }
  type Robot struct {
  }
  func (p *Robot) Drive(from Location, to Location){
    //implements the drive method
  }
  func main(){
    var A Location
    var B Location
    var random_human_driver *Human = new(Human)
    var random_robot_driver *Robot = new(Robot)
    //不管是人还是机器人只要能够开车都能满足你的要求
    TakeRide(A, B, random_human_driver)
    TakeRide(A, B, random_robot_driver)
  }
```
非侵入式接口一个很重要的好处就是去掉了繁杂的继承体系，我们看许大神在《go语言编程》一书中作的总结：
其一， Go语言的标准库，再也不需要绘制类库的继承树图。你一定见过不少C++、 Java、 C# 类库的继承树图。在Go中，类的继承树并无意义，你只需要知道这个类实现了哪些方法，每个方法是啥含义就足够了。 
其二，实现类的时候，只需要关心自己应该提供哪些方法，不用再纠结接口需要拆得多细才 合理。接口由使用方按需定义，而不用事前规划。 
其三，不用为了实现一个接口而导入一个包，因为多引用一个外部的包，就意味着更多的耦 合。接口由使用方按自身需求来定义，使用方无需关心是否有其他模块定义过类似的接口.

## 类型断言

```go
  func TakeRide(from Location, to Location, driver Driver){
    driver.Drive(from, to)
  } 

```

这时候我们可能观察不出这个driver载的是否是哪一位,或者是人,或者是机器人

这里我们就可以使用Type Assert来判断driver的载客

v, ok := driver.(*Human) 
如果driver实际上确实是个Human，那么v就会指向具体的Human类型，ok也会设置为true 
如果driver实际上是个Robot，那么human就会为nil，ok也会设置为false

下面也有一个例子来根据是人或者机器人来做出不同的动作,这里运用go中的switch从而避免了if-else的冗杂

```go
 switch v:= driver.(type) {
  case *Human:
    v.Talk()
  case *Robot:
      v.Sing()
  default:
    fmt.Println("don't know the type but it can drive")
  }
  //当然这需要Human需要能够Talk,Robot能够Sing才可以
   func (p *Human) Talk(){
   fmt.Println("talk something with you")
  }
  func (p *Robot) Sing(){
    fmt.Println("sing a song for you")
  }
```

## 在面向对象中扮演的角色

golang中是没有完整的面向对象思想,没有继承,而多态只能用接口来实现

golang只能模拟继承,也就是组合,golang为我们提供了一些语法糖看起来达到了继承的效果

> 面向对象中一个很重要的基本原则--里氏代换原则(Liskov Substitution Principle LSP)

当你将一个父类的指针指向子类的对象时，golang会毫不吝啬的抛出一个编译错误。

## interface具体实现

我们可以用一张图来观察

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20190812194650.png)

interface在内存上实际由两个成员组成，如下图，tab指向虚表，data则指向实际引用的数据。虚表描绘了实际的类型信息及该接口所需要的方法集

观察itable的结构，首先是描述type信息的一些元数据，然后是满足Stringger接口的函数指针列表（注意，这里不是实际类型Binary的函数指针集哦）。因此我们如果通过接口进行函数调用，实际的操作其实就是s.tab->fun[0](s.data)。是不是和C++的虚表很像？接下来我们要看看golang的虚表和C++的虚表区别在哪里。

golang的实现方式，同C++一样，golang也为每种类型创建了一个方法集，不同的是接口的虚表是在运行时专门生成的。可能细心的同学能够发现为什么要在运行时生成虚表。因为太多了，每一种接口类型和所有满足其接口的实体类型的组合就是其可能的虚表数量，实际上其中的大部分是不需要的，因此golang选择在运行时生成它，例如，当例子中当首次遇见s := Stringer(b)这样的语句时，golang会生成Stringer接口对应于Binary类型的虚表，并将其缓存。

理解了golang的内存结构，再来分析诸如类型断言等情况的效率问题就很容易了，当判定一种类型是否满足某个接口时，golang使用类型的方法集和接口所需要的方法集进行匹配，如果类型的方法集完全包含接口的方法集，则可认为该类型满足该接口。例如某类型有m个方法，某接口有n个方法，则很容易知道这种判定的时间复杂度为O(mXn)，不过可以使用预先排序的方式进行优化，实际的时间复杂度为O(m+n)。




## 参考

https://blog.csdn.net/justaipanda/article/details/43155949

[http://shanks.leanote.com/post/interface%E8%AF%A6%E8%A7%A3](http://shanks.leanote.com/post/interface详解)