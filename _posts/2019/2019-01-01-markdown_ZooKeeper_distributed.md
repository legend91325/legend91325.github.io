---
title: "《ZooKeeper分布式过程协同技术详解》学习笔记"
layout: post
date: 2019-01-01 00:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- zooKeeper
- learning note
category: blog
author: WuKongCoder
description: zookeeper learning
---
>以前博客文章，时间点无法追溯，统一放在 2019-01-01 00:00:00。

#### 第一部分 ZooKeeper概念和基础
Znode 
~~~
/master
/workers 其下每个节点保存系统中一个可用从节点的信息
/tasks  其下每个节点保存所有已经创建并等待从节点执行的任务信息
/assign  其下每个节点保存了分配到某个从节点的一个任务信息，当主节点每分配一个任务就增加一个子节点

持久节点(peisistent)
临时节点(ephemeral)
有序节点(sequential)：
    持久
    临时
~~~
版本机制 保证 setData delete 操作传入版本号为参数，避免操作不一致效果。

ZooKeeper 架构
~~~
独立模式（standalone）
仲裁模式（quorum）
仲裁模式下 复制集群中的所有服务器的数据树（所需立法者的最小数量，选定法定人数准确的大小是非常重要的）
~~~
会话
~~~
一个客户端只打开一个会7话，保证请求FIFO顺序执行。
如果发生网络分区，导致ZooKeeper与客户端隔离，直到显式关闭这个会话（客户端）或者分区问题修复后，
ZooKeeper发送的会话超时。 因为ZooKeeper对声明会话超时负责。

会话超时参数很重要：
假设超时时间为T, 客户端会在T/3的时间未收到任何消息，将向服务器发送心跳消息。
经过2T/3时间后 客户端会开始寻找其他服务器（还剩T/3时间寻找）

ZooKeeper实现中，系统根据每一个更新建立的顺序来分配给事务标识符（zkid），
ZooKeeper确保每一个变化相对于所有其他已执行的更新是完全有序的。
如果一个客户端在位置i观察到一个更新，它就不能连接到只观察到i`<i的服务器上。
~~~
ZooKeeper 实现锁
~~~
通过竞争创建/lock 来获取锁（为了防止无法释放问题，创建临时节点）
~~~

#### 第二部分使用ZooKeeper进行开发

ZooKeeper 管理连接
~~~
当连接发生问题，ZooKeeper客户端会主动尝试重新建立通信。
所以不要关闭会话再启动一个新的会话这样会增加系统的负载，并导致更长事件的中断。
~~~
获得管理权
~~~
create 操作的两种异常要关注(可能已经成功)：
   ConnectionLossException(KeeperException的子类)
   InterruptedException
~~~
回调函数处理
~~~
因为只有一个线程处理回调函数，所以建议一般不要在回调函数中集中操作或者阻塞操作。
一遍后续回调调用可以快速被处理。
~~~
设置元数据
~~~
\workers
\assign
\tasks
\status
~~~
流程
~~~
---> 申请主节点创建(\master) ephemeral
---> 设置元数据(\workers \assign \tasks \status)
---> 其他设置从节点(\workers\work-id【serverid】) ephemeral
---> 应用程序client 队列化新任务(\tasks\task-id【单调递增，保证唯一】) sequential 不是临时节点
---> 主节点给从节点分配任务（\assign\worker-id\task-num）,删除任务(\tasks\task-id)
---> 客户端监视(\status\task-id),任务状态
~~~

单次触发
~~~
匹配监视点条件的第一时间会触发监视点的通知，并且最多通知一次。多个事件会分摊到一个通知上
~~~

Event
~~~
ZooKeeper会话状态（KeeperState）:Disconnected, SyncConnected,AuthFailed
,ConnectedReadOnly,SaslAuthenticated,Expired
事件类型(EventType)：NodeCreated，NodeDeleted,NodeDataChanged,NodechildrenChanged,None

exist : NodeCreated,NodeDeleted,NodeDataChanged
getData:NodeDeleted,NodeDataChanged
getchildren:NodeChildrenChanged
~~~
multiop 原子性执行多个ZooKeeper操作

顺序的保证
~~~
写操作顺序，所有服务器并不需要同事执行这些革新，服务器更可能在不同时间执行状态变化，因为它们以不同速度运行。
但是对应用程序来说，它们感知到的相同更新顺序。ZooKeeper状态要防止通过隐藏通道进行通信。

读操作顺序，不同客户端可能是在不同时间观察到了更新，如果它们在ZooKeeper以外通信，可能读取到错误数据。
要依据监视事件，去读取数据
~~~

监视点羊群效应和可扩展性

故障处理
~~~
Disconnected事件和ConnectionLossException异常的产生的一个典型原因是因为ZooKeeper服务器故障。
当一个进程收到Disconnected事件，在重新连接之前，进程需要挂起群首的操作。

断开重连，客户端会发送监视点列表和最后一只的zxid(最终状态事件戳)，服务器会检查监视点和事件戳，
如果晚于已知的zxid,服务器会触发这个监视点。
Exist操作除外，可能错过事件，与ZooKeeper失联期间，创建又删除，就不会收到事件通知。
~~~

不可恢复故障
~~~
处理不可恢复故障最简单办法就是中止进程并重启，这样可以使进程恢复原状，通过一个新的会话重新初始化自己的状态。

如果简单的重新创建ZooKeeper句柄以覆盖旧的句柄，
这种方式恢复，但是会有问题，旧句柄的信息可能失效，依旧依据那些信息，会产生严重问题。
~~~
