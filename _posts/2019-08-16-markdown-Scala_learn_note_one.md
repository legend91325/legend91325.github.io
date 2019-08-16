---
title: "快学Scala学习笔记【一】"
layout: post
date: 2019-08-16 15:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Scala
- learning note
category: blog
author: wukong
description: Scala learning note
---

## 写在开篇
 好记性不如烂笔头，这几年越发感到学过的看过的东西如果记录下来，有时间回顾一下，可能用不了多久就忘记了。
 另一方面原因，如果不能把看过的东西，简练的描述下来，应该还是看的有点一知半解。
 所以这学习笔记系列，可能样式没有多大讲究，主要是为了监督自己 真正去弄明白所学的东西吧。
 
## 第一章 基础
```val```用来定义无法改变的常量
```var```用来声明需要改变内容的变量

## 第二章 控制结构和函数

在Scala中每个表达式都是有类型的。 公共超类型叫做Any

Unit 写做() 当做标识“无有用之”的占位符 可以当做JAVA中的void

void没有值但是Unit有一个表示"无值"的值。

如果一定要深究，好比空钱包和里面有一张写着没钱的无面值钞票的钱包之间区别。

Scala中```{ }``` 包含一系列表达式，其结果也是一个表达式。块中最后一个表达式的值就是块的值。

for循环中可以有守卫
```Scala
for (i <- 1 to 3; j <- 1 to 3 if i!=3) print ((10 * i +j)+ " " )
//将打印 12 13 21 23 31 32
```

for推导式,推导式生成的集合和与它的第一个生成器是类型兼容的
```Scala
for( c <- "Hello"; i<- 0 to 1 ) yield (c + i).toChar
//将生成HIeflmlmop
for(i <- 0 to 1;c <- "Hello") yield (c + i).toChar
//将生成Vector(H, e, l, l, o, I, f, m, m, p)
```
