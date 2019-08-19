---
title: "Scala 特质"
layout: post
date: 2019-01-01 00:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- scala
- history note
category: blog
author: wukong
description: Scala trait
---

# scala特质 笔记

标签（空格分隔）： scala

---

java 中可以扩展一个超类同事实现任意数量接口，接口只能包含抽象方法。字段都是public static final的。
菱形问题：java中一个类无法同时从两个基类中继承方法。
scala中特质：

特质可以同时有抽象方法和具体方法，而类可以同时实现多个特质
java中不能一个类继承多个类，但可实现多个接口。

特质中为实现的方法，默认就是抽象方法，不用特意声明为abstract。
```scala
trait logger {
    def log(msg: String)    //抽象方法
}
```

在重写特质的抽象方法是不需要给出override关键字。
一个类可以混入多个特质，在第一个特质前使用extends，其他所有特质使用with。
```scala
class ConsoleLogger extends Logger with Cloneable with Serializable
```
解读的方法：`Logger with Cloneable with Serializable 首先是一个整体，然后再由类来扩展`

注意：让特质拥有具体行为存在一个弊端，当特质改变是，所有混入了该特质的类都必须重新编译。

###叠加在一起的特质
为类天剑多个互相调用的特质，**从最后一个特质开始**。这对于需要分阶段加工处理某个值的场景很有用。
```scala
trait TimestampLogger extends Logged{
    override def log(msg:String){
        super.log(new java.util.Date() + "" +msg)
    }
}

trait ShortLogger extends Logged {
    val maxLength = 15
    override def log(msg: String){
        super.log(
        if (msg.length <= maxLength) msg 
        else msg.substring(0,maxLength-3) + "..."
        )
    }
}
```
上述的log方法都将修改故的消息传递给super.log。就特质而言，super.log并不像类那样拥有相同含义。因为特质的层级是不确定的，是根据特质混入的顺序来决定的。**一般来说**，特质从最后一个开始被处理
示例：
```scala
val acct1 = new SavingsAccount with ConsoleLogger with TimestampLogger with ShortLogger
//调用顺序 ShortLogger的log方法先执行，然后它的super.log调用TimestampLogger
val acct2 = new SavingsAccount with ConsoleLogger with ShortLogger with TimestampLogger
//顺序则是TimestampLogger -> ShortLogger -> ConsoleLogger
```
>对特质而言，你无法从源码来判断super.somMethod会执行哪里的方法。确切的方法依赖于使用这些特质的对象或类给出的顺序。这使得super相比在传统的继承关系中要灵活得多。
如果你需要控制具体是哪一个特质的方法被调用，则可以在方括号中给出名称：super[特质名].方法名 这里给出的类型必须是**直接超类型**；你无法使用继承层级中更远的特质或类。

###特质中重新抽象方法
刚刚说了特质中super.方法要在特质被混入后才能确定调用顺序。
下面例子：
```scala
trait Logger {
    def log(msg: String)
}
trait TimestampLogger extends Logger {
    override def log(msg: String){
    //super.log编译器会标记位错误，因为还不确定super
        super.log(new java.util.Date() +""+ msg)
    }
}
```
Scala认为TimestampLogger依旧是抽象的，它需要混入一个具体的log方法。因此，你必须给方法打上abstract关键字以及override关键字。
```scala
abstract override def log(msg: String){
    super.log(new java.util.Date() +""+msg)
}
```

###特质中的具体字段
特质中的字段可以是具体的，也可以是抽象的。如果你给出了初始值，那么字段就是具体的。
>对于特质中的每一个具体字段，使用该特质的类都会获得一个字段与之对应。**这些字段不是被继承的；他们只是简单地被加到了子类当中。**
在JVM中，一个类只能扩展一个超累，因此来自特质的字段不能以相同的方式继承。
你可以把具体的特质字段当做是针对使用该特质的类的“装配指令”。任何通过这种方式被混入的字段都自动成为该类自己的字段。
