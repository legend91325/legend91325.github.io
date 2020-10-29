---
title: "Docker 初识"
layout: post
date: 2020-10-29 00:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Docker
- Big Data
category: blog
author: wukong
description: Docker 
---
> 花了十天业余时间  基本上走完了这篇教程[Docker for beginners](https://docker-curriculum.com/#introduction)，基本上对docker有了初步的认识，打算写下自己的总结，方便加深理解。


### 总结笔记

那些操作命令基本就不记录了，都可以自查，就记录下一些概念和原理吧

镜像
> 镜像是分层的，多个镜像是共享中间层，本质来说镜像就是Linux的文件系统，运行在Linux内核之上的，同时是只读的。

> 镜像技术原理是基于联合文件系统，对镜像层级管理，一个镜像是由基础镜像多层构建而来。


![镜像](https://user-gold-cdn.xitu.io/2019/9/7/16d0c272ef8e8adb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

容器
> 是由镜像创建的，是一个独立于宿主机的隔离进程，有社会教育自己的网络和命名空间，容器是可读可写的，是因为容器就是在镜像上面添加一层可读写层（write/read layer）来实现的。

![容器](https://user-gold-cdn.xitu.io/2019/9/7/16d0c4c920a5c29a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

仓库
> 仓库(Repository)是集中存储镜像的地方，这里有个概念要区分一下，那就是仓库与仓库服务器(Registry)是两回事，像我们上面说的Docker Hub，就是Docker官方提供的一个仓库服务器



DockerFile Docker镜像 Docker容器关系

> Dockerfile 是软件的原材料，Docker 镜像是软件的交付品，而 Docker 容器则可以认为是软件的运行态

![DockerFile Docker镜像 Docker容器](http://blog.daocloud.io/wp-content/uploads/2015/09/allen5.jpg)

> Docker 守护进程会在 Docker 镜像的最上层之上，再添加一个可读写层，容器所有的写操作都会作用到这一层中。而如果 Docker 容器需要写底层 Docker 镜像中的文件，那么此时就会涉及一个叫 Copy－on－Write 的机制，即 aufs 等联合文件系统保证：首先将此文件从 Docker 镜像层中拷贝至最上层的可读写层，然后容器进程再对读写层中的副本进行写操纵。对于容器进程来讲，它只能看到最上层的文件。


### 参考资料
[Docker的三大核心组件：镜像，容器与仓库](https://juejin.im/post/6844903938030845966)

[深入分析 Docker 镜像原理](http://blog.daocloud.io/principle-of-docker-image/)

[Docker存储驱动—Overlay/Overlay2](https://arkingc.github.io/2017/05/05/2017-05-05-docker-filesystem-overlay/)
