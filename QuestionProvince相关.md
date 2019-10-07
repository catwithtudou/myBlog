---
title: QuestionProvince相关
date: 2019-09-29 23:32:06
tags: ["golang","项目反思"]
categories: "项目反思"
---

# QuestionProvince相关

## 随机相关

在写该项目的时候,我有一个功能是需要从某地区中随机抽取部分题目

所以我有两个选择

- 第一个是在数据库中直接随机抽取结果
- 第二个是在程序中生成不重复的随机数来抽取

### mysql随机查询

一般我们知道在sql语句中有`rand()`这一函数

即很容易想到

```mysql
SELECT * FROM `table` ORDER BY RAND() LIMIT 1
```

但是这条语句的查询效率贼慢,测试用15万条数据的表中查询5条随机抽取5条记录就花了8秒以上

通过google,差不多都可以看到用`max(id)*rand()`的方法来随机获取数据

下面两个sql语句是比较常用且效率也比较高的

- 使用`JOIN`来使用相关函数

```mysql
SELECT *
FROM `table` AS t1 JOIN (SELECT ROUND(RAND() * ((SELECT MAX(id) FROM `table`)-(SELECT MIN(id) FROM `table`))+(SELECT MIN(id) FROM `table`)) AS id) AS t2
WHERE t1.id >= t2.id
ORDER BY t1.id LIMIT 1;
```

- 使用`WHERE`来使用相关函数

```mysql
SELECT * FROM `table`
WHERE id >= (SELECT floor( RAND() * ((SELECT MAX(id) FROM `table`)-(SELECT MIN(id) FROM `table`)) + (SELECT MIN(id) FROM `table`))) 
ORDER BY id LIMIT 1;
```

> 最后对这两个语句进行测试,分别查询10次
>
> 使用`JOIN`:0.01530秒
>
> 使用`WHERE`:0.147433秒
>
> 通过对比也可以发现`JOIN`中使用函数的效率要高得多

### 不重复的随机数生成

> 随机数的用途非常广泛，比如取样、产生随机密码等。我们通常遇到的随机数的一个经典应用–洗牌算法。洗牌大家都很熟悉，在打牌时，为了保证下次抓牌的随机性和公平性，我们打完每一局牌都需要重新洗牌。抽象成模型就是把一个列表顺序随机打乱。

我们可能遇到比较多的情况是把一个无序的列表排序成一个有序列表。而洗牌算法（shuffle）则是一个相反的过程，它是把一个有序的列表（当然无序也无所谓）变成一个顺序完全随机的无序的列表。同样需要满足两个条件：

- 不可计算性。即打乱之前不能通过其它任何方式得出随机列表。
- 机会均等性。列表的每个元素出现在任意索引位置的概率是相等的。



下面列举出现在比较常见的三种思路

- 第一种思路:**基于暴力求解**

  - 我们不能保证每次生成的随机数都是不重复的，但是可以在生成随机数之后，判断这个数值是否已经生成过，如果已经生成过了，那就重新生成一个直到没有出现重复的新数值。

  

  这种思路是我们平常最常见的一种思路

  但是我们可以明显地看到
  
  时间复杂度为O(N²) 
  
  空间复杂度是O(N)
  
- 第二种思路:**基于哈希**

  - 每生成一个随机数之后，通过下标的方式来进行判断是否重复，从而保证每次生成的随机数一定是新的

    

  实现这一种思路我们比较容易想到用哈希表来判断是否出现重复数

  由此时间复杂度变为O(N²)降至O(N),而空间复杂度仍然基本保持在O(N)

  但是这种算法也有很明显的缺陷那就是

  - 随着Hashtable中的数值越来越多，重复概率也会越来越高

- 第三种思路:**基于交换思想**

  - 依次遍历列表的所有元素，设当前遍历的数的索引为i，利用随机函数生成器产生1～100的随机数，比如产生88，则交换第i张牌和第88张牌。

  该方案是原地操作（空间复杂度为O(1))，并且避免了位置冲突问题。但是否能够保证每个数的放置满足机会均等性呢？

  我们知道n张牌，利用随机数产生N种情况，要满足N中情况出现概率相等，则必须满足N能够整除n，这样就能给予每个牌以N/n的机会，这是必须满足必要条件。想象下如果N不能整除n，一定不能保证n出现的概率相同。我们可以通过简单的例子解释下原因：

  - 假设我们的随机数能够随机生成1-8共8个数，我们需要得到0/1的随机数，则我们只需要规定生成的偶数为0，奇数为1即可。
  - 假设我们的随机数能够生成1-9共9个数，我们需要得到0/1的随机数，因为9不能被2整除，多出的一个数到底归为0呢，还是1呢。

  我们知道n个不重复的数共有`100!`全排列组合情况，而调用n次随机函数，每次生成1-n之间的整数，共可以产生`n^n`种组合情况。而`n^n`一定不能整除`n!`，想想为什么？

  我们的第二种方案，每次都随机产生1-100的数，重复调用100次，相当于有`100^100`种可能情况，而我们的列表只有`100！`种组合可能，显然不能满足机会均等性。

  我们可以利用第二种方法改进，每次不是产生1~100的随机数，而是1~i的数字，则共有`n!`种情况，即`N=n!`，这也就是经典的`Fisher-Yates_shuffle`算法，大多数编程语言库都使用了该算法实现洗牌算法。

  ```go
  func main(){
  	//随机数生成
  	//交换思想
  	result:=make([]int,11)
  	rand.Seed(time.Now().UnixNano())
  	for i:=0;i<11;i++{
  		result[i]=i
  	}
  	for i:=10;i>0;i--{
  		x:=rand.Intn(i+1)
  		temp:=result[i]
  		result[i]=result[x]
  		result[x]=temp
  	}
  	fmt.Println(result)
  
  }
  ```

  时间和空间复杂度都是O(N)，性能非常高。

  通过巧妙的交换位置的方法，可以确保每次得到的数值一定是不重复的，所以不用去判断是否重复。而且，使用数组来保存序列，比List和Hashtable性能更好。

## Bug相关

### HttpsGet

平常我们可以直接使用http库去发送Get请求

但是如果是https就会出现

`x509 certificate signed by unknown authority`

google搜索到的解决办法绝大部分人的解决方法是设置 `TLSClientConfig` 来跳过 *https* 证书验证

即如下代码

```go
func GetHttps(){
    	tr:=&http.Transport{
		TLSClientConfig:    &tls.Config{InsecureSkipVerify: true},
	}
	client:=&http.Client{Transport:tr}
	body, err := client.Get("https://wx.redrock.team/magicloop/keycenter/public")
}
```

但是这个问题的根源也就是证书的问题

首先我们需要了解一下证书链的概念

证书链其实构成了整个证书的信任基础，证书校验方需要对证书链进行正确的校验。我们可以使用 *Chrome* 看一下 *github.com* 的证书链：

![img](https://icell.io/content/images/2018/12/github_ssl_chain.png)

上图中所展示出来的一个证书链的关系就是 *github.com -> DigiCert SHA2 Extended Validation Server CA -> DigiCert High Assurance EV Root CA* 构成，一整条链分为三种证书类型，分别是服务器实体证书、中间证书和根证书。根据证书链的校验过程如下：

1. **获取证书链**：客户端/浏览器连接一个 https 服务器，服务器会将完整的证书链返回给客户端/浏览器。
2. **校验证书链关系：**客户端/浏览器会确保除了跟证书之外的每个证书的签发者是它上一级证书的使用者，如果不一致则证书校验失败。
3. **迭代签名校验**：客户端/浏览器从服务器实体证书的上一级证书（假设是 B 证书）获取 *public key* 来校验服务器实体证书的签名，成功之后从 B 证书的上一级证书（假设是 C 证书）获取 *public key* 来校验 B 证书的签名，成功之后再继续，依次不断迭代，直到结果为根证书的签发人和使用人一致则校验成功。

而我们遇到的这个问题就是 *https* 网站部署证书的时候并没有正确地部署证书链，在第一步获取证书链的时候出现了问题。但是对于 *Chrome* 浏览器或者别的一些开发语言来说，如果服务器返回的证书链不完整，它们会单独对缺失的证书进行补充，但 golang 并没有执行这个操作，导致整个 *https* 协议握手失败，从而出现了这个问题。

### magicloop

magicloop文档写的很清晰,其验证机制也非常安全也就是标准RSA签名和验签

但是在其进行token的转化,遇到了一个弱智问题

```go
func TranslateToken(token string) (payload, reToken string) {
	token = strings.ReplaceAll(token, "%20", "+")
	result := strings.Split(token, ".")
	return result[0], result[1]
}
```

大家可能已经发现那个`%20`,当时没反应过来是URL-encode,看了好久才发现,其实就是`space`

### import cycle not allowed

因为Golang是不支持循环依赖的，所以编译会得到以下错误输出信息：

```java
import cycle not allowed  
    package test/a 
        imports test/b 
        imports test/a
```

最直接的方法就是寻找你循环依赖的地方,并修改相应结构

其实我感觉这也体现出Golang对于禁止循环依赖这一点,也体现出它对各个依赖之间减少其耦合度的思想

当然除了修改相应结构,你也而已通过定义`interface`也能解决此问题， 还能解决连分包都不能解决的情况， 并且比分包更加简单快捷

## 前端交互相关

### 跨域问题

> 这次和前端交互的时候出现了跨域问题
>
> 所以我Google了一些资料来帮助了解

要了解跨域问题的就首先了解跨域是什么东西导致了跨域

那就是浏览器的同源策略

据我了解，浏览器是从两个方面去做这个同源策略的，一是针对接口的请求，二是针对Dom的查询。试想一下没有这样的限制上述两种动作有什么危险。

#### 没有同源策略限制的接口请求

> 接下来举一个来自[写Bug](https://segmentfault.com/u/t_co_b)作者的栗子
>
> 
>
> 有一个小小的东西叫cookie大家应该知道，一般用来处理登录等场景，目的是让服务端知道谁发出的这次请求。如果你请求了接口进行登录，服务端验证通过后会在响应头加入Set-Cookie字段，然后下次再发请求的时候，浏览器会自动将cookie附加在HTTP请求的头字段Cookie中，服务端就能知道这个用户已经登录过了。知道这个之后，我们来看场景：
> 1.你准备去清空你的购物车，于是打开了买买买网站www.maimaimai.com，然后登录成功，一看，购物车东西这么少，不行，还得买多点。
> 2.你在看有什么东西买的过程中，你的好基友发给你一个链接www.nidongde.com，一脸yin笑地跟你说：“你懂的”，你毫不犹豫打开了。
> 3.你饶有兴致地浏览着www.nidongde.com，谁知这个网站暗地里做了些不可描述的事情！由于没有同源策略的限制，它向www.maimaimai.com发起了请求！聪明的你一定想到上面的话“服务端验证通过后会在响应头加入Set-Cookie字段，然后下次再发请求的时候，浏览器会自动将cookie附加在HTTP请求的头字段Cookie中”，这样一来，这个不法网站就相当于登录了你的账号，可以为所欲为了！如果这不是一个买买买账号，而是你的银行账号，那……

这个就是我们熟知的CSRF攻击方式的一种

其实即使有了同源策略限制,但是我们知道cookie是明文,这个时候就需要服务端设置httpOnly来使得前端无法操纵cookie,若没有该设置，像XSS攻击就可以去获取到cookie[Web安全测试之XSS](https://www.cnblogs.com/TankXiao/archive/2012/03/21/2337194.html)；设置secure，则保证在https的加密通信中传输以防截获。

#### 没有同源策略限制的Dom查询

> 1.有一天你刚睡醒，收到一封邮件，说是你的银行账号有风险，赶紧点进www.yinghang.com改密码。你吓尿了，赶紧点进去，还是熟悉的银行登录界面，你果断输入你的账号密码，登录进去看看钱有没有少了。
> 2.睡眼朦胧的你没看清楚，平时访问的银行网站是www.yinhang.com，而现在访问的是www.yinghang.com，这个钓鱼网站做了什么呢？

```html
// HTML
<iframe name="yinhang" src="www.yinhang.com"></iframe>
// JS
// 由于没有同源策略的限制，钓鱼网站可以直接拿到别的网站的Dom
const iframe = window.frames['yinhang']
const node = iframe.document.getElementById('你输入账号密码的Input')
console.log(`拿到了这个${node}，我还拿不到你刚刚输入的账号密码吗`)

```

由此我们知道

同源策略并不能解决所有的问题也不能避免所有的攻击,但是同源策略是一种浏览器最基本的安全机制,始终可以提高一点攻击成本

#### golang解决方案

下面给出我自己的一种解决方案,那就是设置相关请求头信息来处理跨域请求,并且支持options访问

```go
// 处理跨域请求,支持options访问
func Cors() gin.HandlerFunc {
	return func(c *gin.Context) {
		method := c.Request.Method

		c.Header("Access-Control-Allow-Origin", "*")
		c.Header("Access-Control-Allow-Headers", "Content-Type,AccessToken,X-CSRF-Token, Authorization, Token")
		c.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS")
		c.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers, Content-Type")
		c.Header("Access-Control-Allow-Credentials", "true")

		//放行所有OPTIONS方法
		if method == "OPTIONS" {
			c.AbortWithStatus(http.StatusNoContent)
		}
		// 处理请求
		c.Next()
	}
}

```

### 参数提交

这次在进行交互的时候也出现了参数格式的问题

以前的时候我是默认为x-www-form-urlencoded

但是这次前端使用的是raw

由此我也询问了一下相关参数交互的问题,了解到



## 不足相关

### 实体混乱

- 在项目一般需要定义三种类型结构体
  - 一般的结构体
  - Model层需要的结构体
  - Controll层返回对象的结构体

在写这个项目的时候一直比较混乱应该如何放,虽然按照以前写java的时候会在entity层定义各种对象,但是golang的结构体与对象差别比较大

按我自己的感觉来说,java的对象比较万能,在任何地方都能使用到,而golang的结构体仅仅用在需要它的地方,也就是针对性很强

### 测试相关

- 微信相关

  微信的缓存实属恶心

  有些接口它缓存了之后根本不调接口

  特别入口那一块,如果你之前测试的时候同意过那个重邮小帮手询问是否绑定信息后,之后的话就不会进入前端向后端发送请求的那个接口,因为我是通过那个相当于入口的接口存储用户信息的,所以不进入那个接口我就无法存储信息

- 内网穿透

  因为之前在没放到gitlab时是部署到自己的小机上面,如果可以通过内网穿透的话,直接用本地的公网Ip就可以直接让外地访问,这样可以大大提高测试效率,减少部署流程



## 参考

<https://icell.io/how-to-solve-golang-x509-issue/>

[http://int32bit.me/2014/10/24/%E9%9A%8F%E6%9C%BA%E7%AE%97%E6%B3%95%E5%92%8C%E6%B4%97%E7%89%8C%E7%AE%97%E6%B3%95/](http://int32bit.me/2014/10/24/随机算法和洗牌算法/)

<https://segmentfault.com/a/1190000015597029>