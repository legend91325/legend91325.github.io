---
title: "Mysql 分区 分表相关总结之方案选择"
layout: post
date: 2015-03-05 17:25
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Mysql
- history note
category: blog
author: WuKongCoder
description: Mysql 分区 分表
---


# Mysql 分区 分表相关总结之方案选择

标签（空格分隔）： mysql 

[TOC]

##引述
前段时间项目需要，一直在研究mysql sharding，看了一些这方面的资料，也亲自实验测试了一些数据。在此，做个概括的笔记，方便以后回顾知识，其实大多是借鉴网络上各位前辈的，然后抱着学习态度去实践，积累属于自己的东西。

##拆分策略选择
其实拆分很灵活，有的是**垂直切分**，将一个库拆成两个或多个，将有相关联的表放在一个库里。有的是**水平切分**将数据量大的表按照一定逻辑进行拆分。个人感觉垂直切分的相对来说缓解了IO的瓶颈，而水平切分，目的是减轻了单个表或某些表读写的压力。
我们项目根据个人需求，采用的水平切分，没有去分库。之后要看看需要采用何种的切分了。
了解到的有： 分表、分区、MERGE引擎分表。
###MERGE引擎分表
####简介
先介绍merge表，此方法**只**适用于MyISAM。我数据库的表都是采用InnoDB引擎的，所以首先就被pass了，但是还是在这里简单介绍下吧。
mysql 5.1 手册里的说的

>An alternative to a MERGE table is a partitioned table, which stores partitions of a single table in separate files. Partitioning enables some operations to be performed more efficiently and is not limited to the MyISAM storage engine. 

改变到MERGE引擎表，意味着成为一个被分区的表，这样将单一的表各分区存储在分离的文件中。分区可以使一些操作效率更显著，并且不受MyISAM存储引擎的限制。（蹩脚的英语，各位看官多担待吧。）

**以上应该是使用merge表的主要原因吧。**

####创建使用
能够创建MERGE表的要求，首先是一组数据结构完全相同的表，并且存储引擎为MyISAM。

让我们先创建一个
```mysql
mysql> CREATE TABLE t1 (
    ->    a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
	->    message CHAR(20)) ENGINE=MyISAM;
mysql> CREATE TABLE t2 (
    ->    a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    ->    message CHAR(20)) ENGINE=MyISAM;
mysql> INSERT INTO t1 (message) VALUES ('Testing'),('table'),('t1');
mysql> INSERT INTO t2 (message) VALUES ('Testing'),('table'),('t2');
mysql> CREATE TABLE total (
    ->    a INT NOT NULL AUTO_INCREMENT,
    ->    message CHAR(20), INDEX(a))
    ->    ENGINE=MERGE UNION=(t1,t2) INSERT_METHOD=LAST;
```
之后查询
```mysql
mysql> SELECT * FROM total;
+---+---------+
| a | message |
+---+---------+
| 1 | Testing |
| 2 | table   |
| 3 | t1      |
| 1 | Testing |
| 2 | table   |
| 3 | t2      |
+---+---------+
```
你创建了total表，只是相当于在t1,t2的表的基础上创建的，需要注意的是在单个表中的主键或唯一索引，放在MERGE后的total表中就不能再当唯一索引用了，这点应该比较好理解但还是要说一下的。
同时你可以`drop`或者` ALTER TABLE tbl_name UNION=(...)`改变表的数据集，这样可以让其动态变化，剔除不需要的。


####使用场景
如果你的数据记录呈现一定时间规律，比如每天产生的一些需要记录的日志，可能你只需要最近一个月的或者最近几个月的，这样你可以每天或者一定时间创建一个数据表，当需要查询一段时间的数据，你只要将这段时间的数据表创建
一张总计的MERGE表。这样数据集可以控制在可控的范围呢，不错吧。so easy。

###分表
分表其实想法上很简单，顾名思义就是将现有的一张数据量大的表去拆分。如果数据库的性能瓶颈在几个关键表上，这时你可以将分表列入你考虑的范围。

####遇到的问题
我说说我在实验分表时遇到的问题和相关解决方式

1.**如何去分表**
根据什么策略把现有表中的数据分到多个表中，并且还有考虑到以后的扩展性上。
德问上的这篇讨论可以借鉴下，

- [《mysql 分表，拆分策略都有哪些？各在什么情况下应用？》](http://www.dewen.io/q/696/)
- [《又拍网拆分策略》](http://www.infoq.com/cn/articles/yupoo-partition-database)

是建立一张索引表，用户id与数据库id对应，（这里他将相同结构的表分在了不同的数据库中进一步减少压力，但同时对于数据的同步也需要通过其他手段来解决），其本质也是分表了同时分库了。这么做的好处是便于以后的扩展，但损耗一点性能，因为会多一次查询嘛。

个人想法，这样索引表可能会成为新的瓶颈，除非用户不会一直增长哈。
我的做法属于另一种，写了个算法通过计算某列值，按照一定规律将数据大致均分在每个分表中。至于扩展性，写算法时候考虑进去了以后增加分表数的问题了。
选择哪种策略，是要看自己的表的业务特点了，方法没有绝对的优缺，还是要根据自己的需求选取。

2.**分表之后主键的维护**
分表之前，主键就是自动递增的bigint型。所以主键的格式已经提早被确定了，像什么uuid之类的就被直接pass掉了。
还有想过自己写一个主键生成程序，利用Java 的Atomic原子量特性，但是考虑还需要增加工作量并且高并发下，这里很可能是个隐患。
还有就是通过应用层上管理主键，如redis中有原子性的递增。
网上较有名的策略是[《Ticket Servers: Distributed Unique Primary Keys on the Cheap》](http://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/),
大致意思是使用一张名Tickets64的MyISAM存储引擎表，专门用来存储主键，数据只有一行，用的话通过
```mysql
REPLACE INTO Tickets64 (stub) VALUES ('a');
SELECT LAST_INSERT_ID();
```
来取。并且设置了两个库，相同的方法，只是每次增长的步长不同，防止一个宕掉，还可以稳定运行。
其他较好的文章[ 《数据库分库分表(sharding)系列(二) 全局主键生成策略》](http://blog.csdn.net/bluishglc/article/details/7710738),[《关于主键管理》](http://www.jdon.com/46684),[《分库分表(sharding)后主键全局唯一性的解决方案》](http://www.mysqlab.net/blog/2009/03/mysql-sharding-unique-primary-key-solution/)

2.**动态选择表名**
表分好之后，问题又来了，数据库层我们的项目使用的是Mybatis框架。SQL语句都写在了xml文件中，现在我需要动态的设置表名。
其实设置mybatis本身，就可以解决这个问题
>statementType	STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED

只要把属性`statementType`设置为`STATEMENT`，表名就可以以参数形式传入。传入参数时要以美元符`${columnName}`这样传入参数，至于Statement，PreparedStatement 的区别我想大家应该都能知道的。

另一种解决方式，是使用[《shardbatis插件》](https://code.google.com/p/shardbatis/),它是开源的，可以实现数据水平切分功能，有兴趣的朋友可以了解下。

###分区表
从mysql5.1之后，提供了一种partition引擎的表，看这句
>In effect, different portions of a table are stored as separate tables in different locations.
实际上，一个表的各个部分可以以单独的个体表存储在不同的位置（略微蹩脚）

在我的理解，如果把一张表分区之后，不同分区放在不同磁盘位置上，对整体的读取是否更有益？
####分区表优缺点
这里主要是看的mysql手册，我也就起到了个翻译的作用。
>Partitioning makes it possible to store more data in one table than can be held on a single disk or file system partition.
相比一张表，只能存放在一块硬盘或者文件系统分区内。分区方式让存储更多数据成为了可能。

>Data that loses its usefulness can often be easily removed from a partitioned table by dropping the partition (or partitions) containing only that data. Conversely, the process of adding new data can in some cases be greatly facilitated by adding one or more new partitions for storing specifically that data.
失效的数据通过dropping掉仅仅包含此数据的分区方式，更容易的被移除。反之，通过添加新的分区来存储一些新的数据，这种方式更加容易。

>Some queries can be greatly optimized in virtue of the fact that data satisfying a given WHERE clause can be stored only on one or more partitions, which automatically excludes any remaining partitions from the search. Because partitions can be altered after a partitioned table has been created, you can reorganize your data to enhance frequent queries that may not have been often used when the partitioning scheme was first set up. This ability to exclude non-matching partitions (and thus any rows they contain) is often referred to as partition pruning, and was implemented in MySQL 5.1.6. 
这句翻译起来很吃力，我就说下大致意思吧，当你以某列分区之后，查询语句where中如果可以指定特有分区或者一个范围的话，查询会得到优化。其实也好理解，因为你在where中指定分区，查询就会只去检索你指定的那块分区，其他的数据不会去检索。后部分说的是可以在创建好的分区上修改分区，使其更合理。

>Queries involving aggregate functions such as SUM() and COUNT() can easily be parallelized.
那些聚集函数，比如SUM(),COUNT() 容易被并行处理。（听起来很酷哦）

这两篇文章写的比较不错，[《MySQL分区表的优缺点》](http://adamlu.net/dev/2012/06/benefit-and-limitations-of-mysql-partition/)，[《mysql分区表对分区函数的限制》](http://www.xuebuyuan.com/1111076.html)。
在选择mysql 分区方案时，还有一个需要考虑的，在mysql的bug中有一个关于mysql分区表查询缓存的bug: [《Partitioning + Query Cache》](http://bugs.mysql.com/bug.php?id=65541)，因为这个问题，mysql已经将分区表的查询缓存disable了，无论你是否开启查询缓存，都不会启用查询缓存。如果你在意这点，请慎重选择方案。

以上是关于，mysql三个拆分方案的总结，资料方面都是自己查找的所以不免有些会不准确，如有发现请务必告知，希望与各位共成长~~~。


note:后续还会考虑写个如何去在数据库层实际操作，建立分区分表以及数据导入测试相关的心得。
