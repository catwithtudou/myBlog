---
title: Linux/Ubuntu中端口相关操作
date: 2019-10-12 23:34:54
tags: ["Linux"]
categories: "Linux"
---

# Linux/Ubuntu中端口相关操作

因为在日常开发中,将项目部署入服务器上,经常会遇到关于端口未开放,占用等问题,也会遇见防火墙打开或关闭的问题,所以这里就想整理一下相关操作,方便以后使用

## netstat

该命令是Linux中查看端口情况比较简单的一种

先简单说一下`netstat`命令各个参数说明:

- -t:指明显示TCP端口
- -u:指明显示UDP端口
- -l:仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)
- -p:显示进程的标识符和名称,每一个套接字/端口都属于一个进程
- -n:不进行DNS轮询,直接显示IP

接下来我就介绍一下常用的三个操作

- `netstat -ntlp`(最常用)

  查看当前所有tcp端口使用情况

  > 若占用则直接使用`kill`相关程序

- `netstat -ntulp | grep XX`

  查看指定端口使用情况

-  `netstat -apn`

  查看所有的进程和端口使用情况

## ufw

在ubuntu中有一个强大的防火墙辅助工具`ufw`

使用起来非常的简单简洁

以下我就介绍常用的几个操作

- `sudo ufw enable`

  防火墙的打开

- `sudo ufw reload`

  防火墙的重启

- `ufw allow 8090`

  打开相应的端口

- `ufw status`

  查看本机端口使用的情况

## iptables

`iptables`也是程序员们比较常用的命令,用来定制防火墙相关规则

- `sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT`

  添加规则,开放80tcp端口

- `sudo iptables-save`

  暂时保存规则

- `sudo apt-get install iptables-persistent`

  安装iptables可以将iptables的规则持久化

- `sudo netfilter-persistent save`

  持久保存规则

- `sudo netfilter-persistent reload`

  重启`iptables-persistent`