---
title: golang项目部署至服务器
date: 2019-10-12 23:53:54
tags: ["golang"]
categories: "golang"
---

# golang项目部署至服务器

golang的项目部署到服务器相对于其他语言应该最简单的

因为golang可以生成.exe或二进制文件,所以可以在服务器上直接运行并且不用担心环境的差异,golang的移植性是非常优秀的

接下来,就我们来说说如何部署吧

## Window服务器

window服务器部署非常的简单

只需要一个指令

- ```go
  go build main.go
  ```

  本地编译

  编译后会在同级目录下生成可执行文件

  `main.exe`

直接运行即可

## Linux服务器

前面我们说到golang项目不仅可以生成可执行文件还可以生成二进制文件

以下就有两种方法来部署

### 本地编译(推荐)

在项目中main.go目录下打开控制台

输入以下三个指令

```go
set GOARCH=amd64
set GOOS=linux
go build main.go
```

然后就会在同级目录下生成二进制文件:

`main`

然后直接将二进制文件传输放入linux服务器某个文件夹中

> 可通过Xftp(可与服务器目录连接)工具来进行文件传输,十分友好

最后直接在运行二进制即可:

`./main`

> 需要注意
>
> 你需要看是否有足够的权限执行该二进制
>
> 可以通过`chmod 777 main`赋予权限

如果想要项目在后台执行,即需要用到

`nohup ./main &`

### 服务器编译

服务器上编译的话,你需要配置其Go的环境

即需要配置

- GOPATH
- GOROOT

相关环境配置我这里就不仔细说了,请自行Google

> 部署的环境要与本地一样

配置好相应环境后部署也比较简单无脑

- 项目里所有依赖包也需要导入其GOPATH的`pkg`中
- 项目源码拷贝到GOPATH的`src`中

最后直接在main.go同级目录中输入指令

```go
go build main.go
```

编译后即会生成二进制文件

`main`

执行方法与上述一致

即`./main`,`nohup ./main &`

