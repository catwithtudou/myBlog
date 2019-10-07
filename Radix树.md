---
title: Radix树
date: 2019-09-09 08:39:18
tags: "数据结构"
categories: "数据结构"
---

# Radix树

以下是分析gin框架中使用的Radix树

```go
package gin

import (
	"net/url"
)

//路由树,保存路由所有信息
type MethodTree struct {
	Method string
	Root   *node
}
type methodTrees []MethodTree

//路由树节点
type node struct {
	path        string       //路由相对路径
	indices     string       //路由树索引
	children    []*node      //树子节点
	handlers    HandlerChain //处理函数链
	priority    uint         //处理函数优先级
	ndType      nodeType     //节点类型
	numParams   uint8        //参数总个数
	specialNode bool         //通配符参数节点
	fullPath    string       //最后路径
}

//节点类型
type nodeType uint8

const (
	static nodeType = iota
	root    //根节点
	param   //':'参数节点
	all     //'*'节点
)

//路由树节点值
type nodeValue struct {
	handlers HandlerChain
	params   Params
	fullPath string
	tsr bool  //是否以'/'进行结尾
}

//URL参数值
type Param struct {
	Key   string
	Value string
}
type Params []Param

const MaxParams = 255

//直接获取参数值
func (p Params) Get(name string) (string, bool) {
	for _, entry := range p {
		if entry.Key == name {
			return entry.Value, true
		}
	}
	return "", false
}

//根据其方法名来返回其根节点
func (tree methodTrees) get(method string) *node {
	for _, tree := range tree {
		if tree.method == method {
			return tree.root
		}
	}
	return nil
}

//根据':'和'*'来计算路径中动态参数的个数
func countSpecialParams(path string) uint8 {
	var n uint
	for _, v := range path {
		if v == ':' || v == '*' {
			n++
		}
	}
	//若大于阈值则返回阈值
	if n >= MaxParams {
		return MaxParams
	}
	return uint8(n)
}

//判断是否增加孩子节点的优先级,且若增加则要交换孩子结点索引在父节点之中的位置
func (n *node) incrementChildPri(pos int) int {
	Repos := pos
	n.children[pos].priority++
	pri := n.children[pos].priority
	//根据优先级交换在父节点中孩子节点的位置
	for Repos > 0 && n.children[Repos-1].priority < pri {
		n.children[Repos-1], n.children[Repos] = n.children[Repos], n.children[Repos-1]
		Repos--
	}
	//如果交换了位置则也要交换索引的位置
	if pos != Repos {
		n.indices = n.indices[:Repos] + n.indices[pos:pos+1] + n.indices[Repos:pos] + n.indices[pos+1:]
	}

	return Repos
}

//向路由树中保存路由信息
//此树的构建方法是Trix树的变形,为了存入在URL里面动态参数的信息
//比如当'/support'已经存入头节点,加入路由'/search'
//先判断出相同字符的长度,然后将'/s'分出作为头节点,而将'upport'作为子节点,并且增加头节点的优先级
//然后将新加入的'/search'分成'earch'为的新的孩子节点,且将'upport'和'earch'的第一个字符作为父节点的indices即'ue'
//且父节点的indices会根据孩子节点的优先级来交换位置,而孩子节点的优先级是根据孩子节点在父节点下handler的个数
//如果此父节点的孩子节点为参数节点则要判断加入的路径是否与该参数节点位置冲突,比如'/a/:aa'与'/a/:aaa'冲突
func (n *node) addRoute(path string, handlers HandlerChain) {
	n.priority++
	fullPath := path
	numParams := countSpecialParams(path)
	parentFullPathIndex := 0 //父节点路径的下标
	if len(n.path) > 0 || len(n.children) > 0 {

	run:
		for {
			//更新最大参数的个数
			if numParams > n.numParams {
				n.numParams = numParams
			}
			//寻找到最长相同前缀字符的下标
			var i = 0
			for i < Min(len(path), len(n.path)) && n.path[i] == path[i] {
				i++
			}

			//若符合条件则先分割出当前节点的父子节点
			if i < len(n.path) {
				child := node{
					path:        n.path[i:],
					specialNode: n.specialNode,
					indices:     n.indices,
					children:    n.children,
					priority:    n.priority - 1,
					fullPath:    n.fullPath,
				}

				//更新最大参数个数
				for i := range child.children {
					if child.children[i].numParams > child.numParams {
						child.numParams = child.children[i].numParams
					}
				}

				//更新父节点即'/s'分出作为父节点
				n.children = []*node{&child}
				n.path = path[:i]
				n.handlers = nil
				n.specialNode = false
				n.fullPath = fullPath[:parentFullPathIndex]
			}

			//将新加入的节点更新进父节点中
			if i < len(path) {
				path = path[i:]

				//如果当前父节点的孩子节点为参数节点,则需要与新加入的节点判断是否与此参数节点冲突
				//如将"/usr/:id/info"和"/usr/:name/info",或者"/usr/:name"和"/usr/name",或者
				//"/usr/*"和"/usr/:id"等情况发生
				if n.specialNode {
					parentFullPathIndex += len(n.path) //更新其父节点的下标
					n = n.children[0]                  //动态参数一般为父节点的唯一孩子节点
					n.priority++

					//更新最大参数个数
					if numParams > n.numParams {
						n.numParams = numParams
					}
					numParams--

					//检查是否有完整匹配通配符后的字符,若有问题则直接panic出来
					//例如：/blog/:pp 和 /blog/:ppp，需要检查更长的通配符
					if len(path) >= len(n.path) && n.path == path[:len(n.path)] {
						if len(n.path) >= len(path) || path[len(n.path)] == '/' {
							continue run
						}
					}
					//
					////如':aaa' in new path '/a/:aaa' conflicts with existing wildcard ':aa' in existing prefix '/a/:aa'
					//segPath := path
					//if n.ndType != all {
					//	segPath = strings.SplitN(path, "/", 2)[0]
					//}
					//prefix := fullPath[:strings.Index(fullPath, segPath)] + n.path
					//panic("'" + segPath + "' in new path '" + fullPath +
					//	"' conflicts with existing wildcard '" + n.path +
					//	"' in existing prefix '" + prefix + "'")
				}

				c := path[0]

				//若符合条件则更新此唯一动态参数节点作为父节点以便下面的判断
				if n.ndType == param && c == '/' && len(n.children) == 1 {
					parentFullPathIndex += len(n.path)
					n = n.children[0]
					n.priority++
					continue run
				}

				//若加入的路径分割后头节点符合在父节点的索引中则将此路由加入该索引下的孩子节点
				for i := 0; i < len(n.indices); i++ {
					if c == n.indices[i] {
						parentFullPathIndex += len(n.path) //更新父节点下标为符合索引下的孩子节点
						i = n.incrementChildPri(i)         //更新其新加入的路径的优先级而更新其父节点中的索引位置和孩子节点位置
						n = n.children[i]                  //更新此节点为符合索引下的孩子节点作为父节点
						continue run
					}
				}

				//若不是动态参数节点则将此路由节点插入进合适的父节点位置
				if c != ':' && c != '*' {
					n.indices += string([]byte{c}) //增加该父节点中的索引
					child := &node{
						numParams: numParams,
						fullPath:  fullPath,
					}
					n.children = append(n.children, child)
					n.incrementChildPri(len(n.indices) - 1)
					n = child
				}
				n.insertChild(numParams, path, fullPath, handlers)
				return
			} else if i == len(path) { //若已经相同路由信息则直接panic出来即不用插入
				if n.handlers != nil {
					panic("handlers has already exist in path '" + fullPath + " '")
				}
				n.handlers = handlers
			}
			return
		}
	} else { //此时为空树,则直接插入作为头节点
		n.insertChild(numParams, path, fullPath, handlers)
		n.ndType = root
	}
}

//向树中插入在addRoute函数中已分割好的节点
//若不是动态参数节点则直接插入
//若是动态参数':'通配符,则需要判断以下几点
//是否为合法路径;是否动态参数路径后还有新的孩子节点;是否':'有参数名称
//若是'*'通配符,则需要判断以下几点
//是否为合法路径;是否该路径后还有其他路径;是否通配符前一位为'/'
func (n *node) insertChild(numParams uint8, path string, fullPath string, handlers HandlerChain) {
	var offset = 0 //在路由信息中已存在的最长路径下标

	//若是动态参数,则以通配符分为两种情况
	for i, max := 0, len(path); numParams > 0; i++ {
		c := path[i]
		if c != ':' && c != '*' {
			continue
		}
		end := i + 1
		for end < max && path[end] != '/' { //是否还有其他通配符若有则直接panic若没有则end指向最后一位
			switch path[end] {
			// the wildcard name must not contain ':' and '*'
			case ':', '*':
				panic("there must have only one wildcard per path segment, has: '" + path[i:] + "' in path '" + fullPath + "'")
			default:
				end++
			}
		}
		//若此分割后的节点还有孩子节点,则说明addRoute分割出现错误
		if len(n.children) > 0 {
			panic("wildcard route '" + path[i:end] + "' conflicts with existing children in path '" + fullPath + "'")
		}

		//需要判断通配符后有参数名称
		if end-i < 2 {
			panic("wildcards must be named with a non-empty name in path '" + fullPath + "'")
		}

		//若是通配符':'的情况
		if c == ':'{
			if i>0{ //更新节点path,且更新offset为最后一位
				n.path = path[offset :i]
				offset=i
			}

			//更新其孩子节点
			child :=&node{
				ndType:param,
				numParams:numParams,
				fullPath:fullPath,
			}
			//将孩子节点插入其父节点,且更新其父节点
			n.children=[]*node{child}
			n.specialNode =true
			n=child
			n.priority++
			numParams--


			//若最后一位还有'/'则说明动态参数后还有路径
			//例如'/usr/:name/word'
			//将动态参数其后面的路径插入其父节点
			if end<max{
				n.path = path[offset:end]
				offset = end
				child := &node{
					numParams: numParams,
					priority:  1,
					fullPath:  fullPath,
				}
				n.children = []*node{child}
				n = child
			}
		}else{  //通配符'*'的情况

		    //其'*'后的路径不能添加其他路径若有其他参数或不是最后一位则直接panic
			if end != max || numParams > 1 {
				panic("'*' routes are only allowed at the end of the path in path '" + fullPath + "'")
			}
			//若'*'其后还有字符也直接panic例如"/usr/*/"
			if len(n.path) > 0 && n.path[len(n.path)-1] == '/' {
				panic(" '*' conflicts with existing handle for the path segment root in path '" + fullPath + "'")
			}
			//'*'前一位根据合法路径则必是'/',若不是则直接panic
			i--
			if path[i] != '/' {
				panic("no / before '*' in path '" + fullPath + "'")
			}
			//更新路由为其'/*'前面的路由
			n.path = path[offset:i]

			//插入其节点与上面':'的情况相同
			child := &node{
				specialNode: true,
				ndType:     all,
				numParams: 1,
				fullPath:  fullPath,
			}
			n.children = []*node{child}  //更新其孩子节点
			n.indices = string(path[i])
			n = child
			n.priority++ //增加优先级

			//更新第二个节点插入其父节点
			child = &node{
				path:      path[i:],
				ndType:     all,
				numParams: 1,
				handlers:  handlers,
				priority:  1,
				fullPath:  fullPath,
			}
			n.children = []*node{child}
			return
		}
	}

	//若不是动态参数节点则直接插入
	n.path = path[offset:]
	n.handlers = handlers
	n.fullPath = fullPath
}


//根据请求方法和路径即uri来在树中寻找相应的handler处理函数
//其中tsr判断就是是否以'/'结尾的uri然后根据tsr来进行是否以模糊匹配来处理该路径
//而unescape就是判断是否需要QueryEscape解码
func (n *node)getValue(path string,pa Params,unescape bool)(value nodeValue){
	      value.params=pa
run:
	for {

		//若当前路径长度比当前节点的路径长度大,则继续进行寻找
		if len(path) > len(n.path) {
			//若此当前路径满足其当前节点的路径则继续向下寻找
			if path[:len(n.path)] == n.path {
				//将当前路径更新为当前节点路径进行向下的路径
				path = path[len(n.path):]
				//如果不是动态参数节点类型,则直接根据当前节点的索引进行向下寻找
				//若找到则以该索引代表的孩子节点进行下一个循环继续进行匹配
				if !n.specialNode {
					c := path[0]
					for i := 0; i < len(n.indices); i++ {
						if c == n.indices[i] {
							n = n.children[i]
							continue run
						}
					}
					//若没有索引则说明匹配未成功则返回tsr
					value.tsr = path == "/" && n.handlers != nil
					return
				}

				//若是动态参数类型,则根据param和all来分情况进行判断
				n = n.children[0]
				switch n.ndType {
				case param:
					//将end变为动态uri的最后一项
					end := 0
					for end < len(path) && path[end] != '/' {
						end++
					}

					//若空间不足则为返回value参数重新分配空间
					if cap(value.params) < int(n.numParams) {
						value.params = make(Params, 0, n.numParams)
					}
					i := len(value.params)
					value.params = value.params[:i+1] // 为返回值分配预空间
					value.params[i].Key = n.path[1:]  //若匹配成功则返回其动态参数部分路径
					val := path[:end]
					if unescape {  //如果需要进行QueryUnescape转码,若不需要则直接进行赋值
						var err error
						if value.params[i].Value, err = url.QueryUnescape(val); err != nil {
							value.params[i].Value = val
						}
					} else {
						value.params[i].Value = val
					}

					//如果还有动态参数则更新当前路径和当前节点的孩子节点进行下一次循环
					if end < len(path) {
						if len(n.children) > 0 {
							path = path[end:]
							n = n.children[0]
							continue run
						}

						//若后面没有路径即没有节点则返回tsr
						value.tsr = len(path) == end+1
						return
					}

					//将路由处理函数且路由路径赋值给返回值
					if value.handlers = n.handlers; value.handlers != nil {
						value.fullPath = n.fullPath
						return
					}

					//若该节点还有子节点说明必定是'/'则需要返回tsr
					if len(n.children) == 1 {
						n = n.children[0]
						value.tsr = n.path == "/" && n.handlers != nil
					}
					return
				case all:  //若为通配符'*'情况

					//若是参数空间不足则重新进行分配
					if cap(value.params) < int(n.numParams) {
						value.params = make(Params, 0, n.numParams)
					}

					//'/*'后面的所有路径都需要返回,其他情况与上面类似,匹配到后进行赋值
					i := len(value.params)
					value.params = value.params[:i+1]
					value.params[i].Key = n.path[2:]
					if unescape {
						var err error
						if value.params[i].Value, err = url.QueryUnescape(path); err != nil {
							value.params[i].Value = path // fallback, in case of error
						}
					} else {
						value.params[i].Value = path
					}
					value.handlers = n.handlers
					value.fullPath = n.fullPath
					return
				default:
					panic("invalid node type")
				}
			}
		} else if path == n.path {	//若当前路径恰好与节点路径相同则直接匹配成功

			//直接进行赋值
			if value.handlers = n.handlers; value.handlers != nil {
				value.fullPath = n.fullPath
				return
			}

			//若是"/"路径不是根结点并且是参数节点且返回tsr
			if path == "/" && n.specialNode && n.ndType != root {
				value.tsr = true
				return
			}

			//若当前节点handler处理函数为空若索引中是'/'则说明需要返回tsr
			for i := 0; i < len(n.indices); i++ {
				if n.indices[i] == '/' {
					n = n.children[i]
					value.tsr = (len(n.path) == 1 && n.handlers != nil) ||
						(n.ndType == all && n.children[0].handlers != nil)
					return
				}
			}

			return
		}

		//若当前路径为"/"且满足当前节点路径第一位则需要返回tsr
		value.tsr = (path == "/") ||
			(len(n.path) == len(path)+1 && n.path[len(path)] == '/' &&
				path == n.path[:len(n.path)-1] && n.handlers != nil)
		return
	}
}




```