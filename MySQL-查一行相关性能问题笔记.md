---
title: MySQL-查一行相关性能问题笔记
date: 2020-02-13 09:45:23
tags: ["MySQL"]
categories: "MySQL"
---

## MySQL-查一行相关性能问题笔记

一般情况下,如果我们谈到查询性能优化的情况时,首先我们会想到一些复杂的语句并且想到查询需要返回大量的数据。但有些情况下，“查一行”也会执行得相当慢。

> 需要说明的是，如果MySQL数据库本身就有很大的压力，导致数据库服务器CPU占用率很高或ioutil（IO利用率）很高，这种情况下所有语句的执行都有可能变慢，不属于我们今天的讨论范围。

## 第一类：查询长时间不返回

首先执行下面SQL语句

```mysq
mysql> select * from t where id=1;
```

假设，现在查询结果长时间不返回。

一般碰到这种情况，大概率是表t被锁住了。所以接下来分析原因的时候，一般是首先执行`show processlist`命令查看当前语句处于什么状态，然后我们再针对每种状态分析它们产生的原因、如何复现以及如何处理。

### 等MDL锁

假如我们使用`show processlist`命令查看到下图所示的结果

![](http://img.zhengyua.cn/img/20200213092738.png)

看到`Waiting for table metadata lock`表示的是**现在有一个线程正在表t上请求或者持有MDL写锁，把select语句堵住了。**

下面给出简单的复现步骤。

![](http://img.zhengyua.cn/img/20200213092949.png)

这类问题的处理方式，就锁找到谁持有MDL写锁，然后把它kill掉。

但是，由于在``show processlist`的结果里面，session A的Command列是“Sleep”，导致查找起来很不方便。不过有了`performance_schema`和`sys`系统库以后，就方便多了。

> MySQL启动时需要设置performance_schema=on，相比于设置为off会有10%左右的性能损失。

通过查询`sys.schema_table_lock_wait`这张表，我们就可以直接找出造成阻塞的process id，把这个连接用kill 命令断开即可。

```mysql
mysql> select blocking_pid from sys.schema_table_lock_waits;
```

### 等flush

接下来，我们再来看另一种查询被堵住的情况。

我们执行下列SQL语句

```mysql
mysql> select * from information_schema.processlist where id=1=6;
```

查询结果如下![](http://img.zhengyua.cn/img/20200213093400.png)

`Waiting for table flush`状态表示的是**现在有一个线程正要对表t做flush操作**。

> MySQL里面对表做flush操作的用法，一般有以下两个：
>
> `flush tables t with read lock;`
> `flush tables with read lock;`

复现步骤如下

![](http://img.zhengyua.cn/img/20200213093511.png)

分析结果如下

![](http://img.zhengyua.cn/img/20200213093527.png)

最后我们只需要kill掉Session A即可

### 等行锁

上述都是表级锁，现在我们来看处于引擎里面的行级锁

```mysql
mysql> select * from t where id=1 lock in share mode; 
```

复现步骤和现场如下

![](http://img.zhengyua.cn/img/20200213093742.png)

![](http://img.zhengyua.cn/img/20200213093748.png)

查询谁占着这个写锁，可以通过sys.innodb_lock_waits表查到，查询方法如下

```mysq
mysql> select * from t sys.innodb_lock_waits where locked_table=`'test'.'t'`\G
```

![](http://img.zhengyua.cn/img/20200213093921.png)

可以看到，这个信息很全，4号线程是造成堵塞的罪魁祸首。我们只需要`KILL QUERY 4`或`KILL 4`。

> 不过，这里不应该显示“KILL QUERY 4”。这个命令表示停止4号线程当前正在执行的语句，而这个方法其实是没有用的。因为占有行锁的是update语句，这个语句已经是之前执行完成了的，现在执行KILL QUERY，无法让这个事务去掉id=1上的行锁。
>
> 实际上，KILL 4才有效，也就是说直接断开这个连接。这里隐含的一个逻辑就是，**连接被断开的时候，会自动回滚这个连接里面正在执行的线程，也就释放了id=1上的行锁。**

## 第二类：查询慢

经过了锁，再来看另外一些查询慢的例子。

如下面这一句SQL语句

```mysql
mysql> select * from t where c=50000 limit 1;
```

我们知道首先上述语句会因为c不是索引字段而扫描5万行。扫描行数多，所以执行慢。

虽然可能查询语句时间太短可能不满足线上慢查询的标准，但是**坏查询不一定是慢查询**。我们这个例子里面只有10万行记录，数据量大起来的话，执行时间就线性涨上去了。

我们再来看一个只扫描一行，但是执行很慢的语句。

```mysql
mysql> select * from t where id=1；
```

作为确认我们可以查看慢日志。

> 注意，这里为了把所有语句记录到slow log里，我在连接后先执行了 set long_query_time=0，将慢查询日志的时间阈值设置为0。

![](http://img.zhengyua.cn/img/20200213094503.png)

可以看到查询一行时间却长达800毫秒。

往下面看slow log可以看到下一个语句`select * from t where id=1 lock in share mode`,执行时扫描行数为1行但执行时间是0.2毫秒。

然后我们再看查询结果。

![](http://img.zhengyua.cn/img/20200213094711.png)

到这里我们应该可以上述语句结果出现的原因。

我们来看复现步骤。

![](http://img.zhengyua.cn/img/20200213094758.png)![](http://img.zhengyua.cn/img/20200213094825.png)

这里我们就可以明显看到是**当前读和一致性读的区别**。

## 问题

我们在举例加锁读的时候，用的是这个语句，`select * from t where id=1 lock in share mode`。由于id上有索引，所以可以直接定位到id=1这一行，因此读锁也是只加在了这一行上。

但如果是下面的SQL语句，

```mysql
begin;
select * from t where c=5 for update;
commit;
```

这个语句序列是怎么加锁的呢？加的锁又是什么时候释放呢？

### 参考

在 **Read Committed 隔离级别**下，会锁上聚簇索引中的所有记录；
在 **Repeatable Read 隔离级别**下，会锁上聚簇索引中的所有记录，并且会锁上聚簇索引内的所有 GAP；
在上面两个隔离级别的情况下，如果设置了 innodb_locks_unsafe_for_binlog 开启 semi-consistent read 的话，对于不满足查询条件的记录，MySQL 会提前放锁，不过加锁的过程是不可避免的。