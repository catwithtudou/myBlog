---
title: MySQL-orderby笔记
date: 2020-02-02 17:23:21
tags: ["MySQL"]
categories: "MySQL"
---

# MySQL-orderby笔记

在日常开发中我们经常会使用到`order by`这一字段,即根据指定的字段排序来显示结果。

以我们前面举例用过的市民表为例，假设查询某一个城市的所有人名字并根据姓名排序返回前1000个人的姓名和年龄。

我们很容易会想到下面的这一条SQL语句：

```mysql
select city,name,age from t where city='杭州' order by name limit 1000  ;
```

今天我们就来了解关于这一个语句是怎么执行的，以及有什么参数会影响执行的行为。

## 全字段排序

为了避免全表扫描，我们需要在`city`字段加上索引，然后我们用`explain`来查看这个语句执行情况

![](http://img.zhengyua.cn/img/20200202170316.png)

我们可以看到`Extra`字段中有一句“Using filesort”，这句表示的就是需要排序，MySQL会给每个线程分配一块内存用于排序，称为`sort_buffer`。

我们再来看一下`city`这个索引的示意图

![](http://img.zhengyua.cn/img/20200202170507.png)

从图中我们可以看到满足`city=‘杭州’`条件的行是从**ID_X到ID_(X+N)**的这些记录

通常情况下，这个语句执行流程如下所示：

1. 初始化sort_buffer，确定放入name、city、age这三个字段；
2. 从索引city找到第一个满足city='杭州’条件的主键id，也就是图中的ID_X；
3. 到主键id索引取出整行，取name、city、age三个字段的值，存入sort_buffer中；
4. 从索引city取下一个记录的主键id；
5. 重复步骤3、4直到city的值不满足查询条件为止，对应的主键id也就是图中的ID_Y；
6. 对sort_buffer中的数据按照字段name做快速排序；
7. 按照排序结果取前1000行返回给客户端。

> 这个排序过程暂且称为全字段排序

![](http://img.zhengyua.cn/img/20200202170759.png)

- 图中按name排序操作可能在内存中完成，也可能需要使用外部排序，这取决于**排序所需的内存和参数`sort_buffer_size`**

> sort_buffer_size，就是MySQL为排序开辟的内存（sort_buffer）的大小。如果要排序的数据量小于sort_buffer_size，排序就在内存中完成。但如果排序数据量太大，内存放不下，则不得不利用**磁盘临时文件辅助**排序。

我们可以使用下面介绍的方法，来确定一个排序语句是否使用了临时文件。

```mysql
/* 打开optimizer_trace，只对本线程有效 */
SET optimizer_trace='enabled=on'; 

/* @a保存Innodb_rows_read的初始值 */
select VARIABLE_VALUE into @a from  performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 执行语句 */
select city, name,age from t where city='杭州' order by name limit 1000; 

/* 查看 OPTIMIZER_TRACE 输出 */
SELECT * FROM `information_schema`.`OPTIMIZER_TRACE`\G

/* @b保存Innodb_rows_read的当前值 */
select VARIABLE_VALUE into @b from performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 计算Innodb_rows_read差值 */
select @b-@a;
```

这个方法就是通过查看`OPTIMIZER_TRACE`的结果来确认的，你可以从`number_of_tmp_files`中看到是否使用了临时文件。

> number_of_tmp_files表示的是，排序过程中使用的临时文件数。

你一定奇怪，为什么需要12个文件？内存放不下时，就需要使用外部排序，外部排序一般使用**归并排序算法**。

![](http://img.zhengyua.cn/img/20200202171105.png)

我们的示例表中有4000条满足``city='杭州’``的记录，所以你可以看到 `examined_rows=4000`，表示参与排序的行数是4000行。

`sort_mode` 里面的``packed_additional_fields`的意思是，排序过程对字符串做了“紧凑”处理。即使`name`字段的定义是`varchar(16)``，在排序过程中还是要按照实际长度来分配空间的。

同时，最后一个查询语句``select @b-@a `的返回结果是4000，表示整个执行过程只扫描了4000行。

> 这里需要注意的是，为了避免对结论造成干扰，我把``internal_tmp_disk_storage_engine`设置成MyISAM。否则，`select @b-@a`的结果会显示为4001。
>
> 这是因为查询``OPTIMIZER_TRACE`这个表时，需要用到临时表，而`internal_tmp_disk_storage_engine`的默认值是InnoDB。如果使用的是InnoDB引擎的话，把数据从临时表取出来的时候，会让`Innodb_rows_read`的值加1。

## rowid排序

上面的算法过程总结下来就是**对原表的数据读了一遍，剩下的操作都是在sort_buffer和临时文件中执行**

但这个算法不可避免的问题就死若查询要返回的字段很多，那么`sort_buffer`里面要放的字段数太多，内存里能够同时放下的行数很少，要分成多个临时文件，排序的性能会很差。

所以如果单行很大，这个方法效率就不够好。

那么，**如果MySQL认为排序的单行长度太大会怎么做？**

接下来，可以通过修改一个参数，让MySQL采用另外一种算法

```mysql
SET max_length_for_sort_data = 16;
#max_length_for_sort_data，是MySQL中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，MySQL就认为单行太大，要换一个算法。
```

要查询的字段长度超过了设置值，那么新的算法放入`soft_buffer`字段，只有要排序的列（即`name`字段）和主键id。

但这时，排序的结果就因为少了city和age字段的值，不能直接返回，整个执行流程就变成如下所下所示的样子：

1. 初始化sort_buffer，确定放入两个字段，即name和id；
2. 从索引city找到第一个满足city='杭州’条件的主键id，也就是图中的ID_X；
3. 到主键id索引取出整行，取name、id这两个字段，存入sort_buffer中；
4. 从索引city取下一个记录的主键id；
5. 重复步骤3、4直到不满足city='杭州’条件为止，也就是图中的ID_Y；
6. 对sort_buffer中的数据按照字段name进行排序；
7. 遍历排序结果，取前1000行，并按照id的值回到原表中取出city、name和age三个字段返回给客户端。

**这个执行流程的示意图如下，我把它称为rowid排序。**

![](http://img.zhengyua.cn/img/20200202172001.png)

对比全字段排序，我们可以发现排序过程中少了不相关的两个字段，而等排序结果出来后再通过主键id回到表中取出那两个字段。

> 需要说明的是，最后的“结果集”是一个逻辑概念，实际上MySQL服务端从排序后的sort_buffer中依次取出id，然后到原表查到city、name和age这三个字段的结果，**不需要在服务端再耗费内存存储结果，是直接返回给客户端的。**

现在，我们就来看看结果有什么不同。

首先，图中的`examined_rows`的值还是4000，表示用于排序的数据是4000行。但是``select @b-@a`这个语句的值变成5000了。

因为这时候除了排序过程外，在排序完成后，还要根据id去原表取值。由于语句是``limit 1000`，因此会多读1000行。

![](http://img.zhengyua.cn/img/20200202172206.png)

从``OPTIMIZER_TRACE`的结果中，你还能看到另外两个信息也变了。

- `sort_mode`变成了**<sort_key, rowid>**，表示参与排序的只有name和id这两个字段。
- `number_of_tmp_files`变成10了，需要排序的总数据量就变小了，需要的临时文件也相应地变少了。

## 全字段排序 VS rowid排序

我们来分析一下，从这两个执行流程里，还能得出什么结论。

- 如果MySQL实在是担心**排序内存太小，会影响排序效率，才会采用rowid排序算法**，这样排序过程中一次可以排序更多行，但是需要再回到原表去取数据。
- 如果MySQL认为**内存足够大，会优先选择全字段排序**，把需要的字段都放到sort_buffer中，这样排序后就会直接从内存里面返回查询结果了，不用再回到原表去取数据。

这也就体现了MySQL的一个设计思想：**如果内存够，就要多利用内存，尽量减少磁盘访问。**

对于InnoDB表来说，**rowid排序会要求回表多造成磁盘读，因此不会被优先选择。**

而我们再回到这个假设本身，如果name字段本身就是递增排序的话，其实就不用再排序了。即MySQL之所以需要生成临时表，并且在临时表上做排序操作，**其原因是原来的数据都是无序的。**

所以，我们可以在这个市民表上创建一个city和name的联合索引，对应的SQL语句是：

```mysql
alter table t add index city_user(city, name);
```

作为与city索引的对比，我们来看看这个索引的示意图。

![](http://img.zhengyua.cn/img/20200202172615.png)

这样整个查询过程的流程就变成了：

1. 从索引(city,name)找到第一个满足city='杭州’条件的主键id；
2. 到主键id索引取出整行，取name、city、age三个字段的值，作为结果集的一部分直接返回；
3. 从索引(city,name)取下一个记录主键id；
4. 重复步骤2、3，直到查到第1000条记录，或者是不满足city='杭州’条件时循环结束。

可以看到，这个查询过程不需要临时表，也不需要排序。接下来，我们用explain的结果来印证一下。

![](http://img.zhengyua.cn/img/20200202172641.png)

从图中可以看到，`Extra字段`中没有``Using filesort`了，也就是不需要排序了，且因为联合索引本身有序，也不用扫描4000行而是只需要找出满足条件的1000行记录就可以退出了。

既然说到这里了，我们再往前讨论，**这个语句的执行流程有没有可能进一步简化呢？**

那就是考虑**覆盖索引**。

> 覆盖索引是指，索引上的信息足够满足查询请求，不需要再回到主键索引上去取数据。

按照覆盖索引的概念，我们可以再优化一下这个查询语句的执行流程。

针对这个查询，我们可以创建一个city、name和age的联合索引，对应的SQL语句就是：

```mysql
alter table t add index city_user_age(city, name, age);
```

这时，对于city字段的值相同的行来说，还是按照name字段的值递增排序的，此时的查询语句也就不再需要排序了。这样整个查询语句的执行流程就变成了：

1. 从索引(city,name,age)找到第一个满足city='杭州’条件的记录，取出其中的city、name和age这三个字段的值，作为结果集的一部分直接返回；
2. 从索引(city,name,age)取下一个记录，同样取出这三个字段的值，作为结果集的一部分直接返回；
3. 重复执行步骤2，直到查到第1000条记录，或者是不满足city='杭州’条件时循环结束。

![](http://img.zhengyua.cn/img/20200202172941.png)

然后，我们再来看看explain的结果。

![](http://img.zhengyua.cn/img/20200202172930.png)

可以看到通过索引，性能回快很多。

当然，这里并不是说要为了每个查询能用上覆盖索引，就要把语句中涉及的字段都建上联合索引，毕竟索引还是有维护代价的。这是一个需要权衡的决定。

## 问题

假设你的表里面已经有了city_name(city, name)这个联合索引，然后你要查杭州和苏州两个城市中所有的市民的姓名，并且按名字排序，显示前100条记录。如果SQL查询语句是这么写的 ：

```mysql
mysql> select * from t where city in ('杭州',"苏州") order by name limit 100;
```

>  那么，这个语句执行的时候会有排序过程吗，为什么？

虽然有(city,name)联合索引，对于单个city内部，name是递增的。但是由于这条SQL语句不是要单独地查一个city的值，而是同时查了"杭州"和" 苏州 "两个城市，因此所有满足条件的name就不是递增的了。也就是说，这条SQL语句需要排序。

> 如果业务端代码由你来开发，需要实现一个在数据库端不需要排序的方案，你会怎么实现呢？

这里，我们要用到(city,name)联合索引的特性，把这一条语句拆成两条语句，执行流程如下：

1. 执行``select * from t where city=“杭州” order by name limit 100;`` 这个语句是不需要排序的，客户端用一个长度为100的内存数组A保存结果。
2. 执行``select * from t where city=“苏州” order by name limit 100;`` 用相同的方法，假设结果被存进了内存数组B。
3. 现在A和B是两个有序数组，然后你可以用归并排序的思想，得到name最小的前100值，就是我们需要的结果了。

> 进一步地，如果有分页需求，要显示第101页，也就是说语句最后要改成 “limit 10000,100”， 你的实现方法又会是什么呢？

如果把这条SQL语句里“limit 100”改成“limit 10000,100”的话，处理方式其实也差不多，即：要把上面的两条语句改成写：

```mysql
select * from t where city="杭州" order by name limit 10100; 
 select * from t where city="苏州" order by name limit 10100;
```

这时候数据量较大，可以同时起两个连接一行行读结果，用归并排序算法拿到这两个结果集里，按顺序取第10001~10100的name值，就是需要的结果了。

当然这个方案有一个明显的损失，就是从数据库返回给客户端的数据量变大了。

所以，如果数据的单行比较大的话，可以考虑把这两条SQL语句改成下面这种写法：

```mysql
select id,name from t where city="杭州" order by name limit 10100; 
select id,name from t where city="苏州" order by name limit 10100;
```

然后，再用归并排序的方法取得按name顺序第10001~10100的name、id的值，然后拿着这100个id到数据库中去查出所有记录。