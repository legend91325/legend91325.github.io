---
title: "快学Scala学习笔记"
layout: post
date: 2019-08-16 15:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- scala
- learning note
category: blog
author: wukong
description: Scala learning note
---

## 写在开篇
 好记性不如烂笔头，这几年越发感到学过的看过的东西如果记录下来，有时间回顾一下，可能用不了多久就忘记了。
 另一方面原因，如果不能把看过的东西，简练的描述下来，应该还是看的有点一知半解。
 所以这学习笔记系列，可能样式没有多大讲究，主要是为了监督自己 真正去弄明白所学的东西吧。
 
### 第一章 基础

val用来定义无法改变的常量
var用来声明需要改变内容的变量


### 第二章 控制结构和函数

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

Scala对于不返回值的函数有特殊的表示法。如果函数体包含在花括号当中单没有前面的=号，那么返回类型就是Unit。 这样的函数被称做过程。
由于过程不返回值，我们可以省略=号
```scala
def box(s: String) {
    var border = "-"*s.length+"--\n"
    println(border +"|"+s+"|\n"+border)
}
//如果不喜欢简明的写法，可以变成
def box(s:String): Unit={  
...
}
```

当val 被声明为lazy时，它的初始化将被推迟，知道我们首次使用它。
你可以把懒值当做是介于val和def的中间状态。对比如下定义：
```scala
//在words被定义时即被取值
val words = scala.io.Source.fromFile("/usr/share/dict/words").mkString
//在words被首次使用时取值
lazy val words = scala.io.Source.fromFile("/user/share/dict/words").mkString
//在每一次words被使用时取值
def words = scala.io.Source.fromFile("/user/share/dict/words").mkString
```
>懒值并不是没有额外开销，每次访问，都会有一个方法被调用，这个方法将会以线程安全的方式检查该值是否已被初始化。

Scala没有受检异常，你不需要声明函数或者方法可能会抛出某种异常
throw表达式有特殊的类型Nothing。这在if/else表达式中很有用。如果一个分支的类型是Nothing，那么if/else表达式的类型就是另一个分支的类型。

### 第三章 数组相关操作

定长数组Array 变长数组ArrayBuffer

数组转换：
```scala
val a = Array(1,2,3,11)
val result = for (elem <- a if elem%2 == 0) yield 2*elem
```
for(...)yield循环 创建了一个类型与原始集合相同的新集合，原始集合并没有受到影响。
另一种风格实现
```scala
a.filter(_%2==0).map(2*_)
//结果完全相同，只是实现的风格不同
```

多维数组
你可以创建不规则数组，每一行的长度各不相同：
```scala
val triangle = new Array[Array[Int]](10)
for(i <- until triangle.length)
    triangle(i) = new Array[Int](i+1)
```

### 第四章 映射与元组

构造映射
```scala
//构造不可变Map[String,Int]
val scores = Map("alice"->10,"bob"->3)
//可变Map
val scores = scala.collection.mutable.Map("alice"->10,"bob"->3)
//空映射
val scores = new scala.collection.mutable.HashMap
```

获取映射中的值
```scala
//不存在键则会抛出异常
val bobscore = scores("bob")
//默认值
scores.getOrElse("bob",0)
//返回Option 要么Some类型（键对应的值），要么None
scores.get("bob")
```

更新操作(可变映射)
```scala
//更新或者增加
scores("bob") = 10
//批量操作
scores += ("bob" -> 10,"fred"->7)
//移除某个键
scores -= "bob"
```
不可变操作(不能更新，但是可以获取新映射)：
```scala
val newScores = scores + ("bob" -> 10,"fred"->7)
```
>你可能觉的这样不停创建新映射效率很低，不过事实并非如此。老的和新的映射共享大部分结构。

元组
>映射是键值对偶的集合。对偶是元组(tuple)的最简单形态-----元组是不同类型的值的聚集。

```scala
val t = (1,3.14,"fred")
//可以用 _1,_2,_3这种方式访问
val second = t._2
//匹配模式 获取
val (first, second, _) = t
```

### 第五章 类
Scala中，类并不声明public，Scala源文件中可以包含多个类，所有这些类都具有公开可见性。
调用无参方法，你可以写上圆括号，也可以不写。
原则上，对于改值器方法（改变对象状态的方法）使用()，对于取值器方法（不会改变对象状态的方法）去掉()是不错的风格。

- 如果字段是私有的，则getter和setter方法也是私有的。
- 如果字段是val，则只有getter方法被生成
- 如果你不需要任何getter或者setter，可以将字段声明为private[this]

对于类私有的字段，Scala生成私有的getter和setter方法。但对于对象私有的字段，Scala根本不会生成getter或setter方法。

标注@BeanProperty，自动生成Java的getXX/setXX方法。

辅助构造器
1. 辅助构造器的名称为this。
2. 每一个辅助构造器都必须以一个对先前已定义的其他辅助狗扎起或主构造器的调用开始。

主构造器
在Scala中，每个类都有主构造器。主构造器并不以this方法定义，而是与类定义交织在一起。
如果没有显示仪定义，则自动有用一个无参的主构造器。
1. 主构造器的参数直接放置在类名之后。
2. 主构造器会执行类的定义中的所有语句。

>如果主构造器的表示法让你困惑，你不需要使用它。你只要按照常规的做法提供一个或多个辅助构造器即可，不过要记得调用this()，如果你不和其他辅助构造器串接的话。
话虽如此，许多程序员还是喜欢主构造器这种精简的写法，Martin Odersky 建议这样来看待主构造器：在Scala中，类也接收参数，就像方法一样。

嵌套类，在Scala中你几乎可以在任何语法结构中内嵌任何语法结构。
```scala
//scala中每个NetWork实例都有她自己的Member类
class NetWork {
   class Member(val name:String){
    val contacts = new ArrayBuffer[Member]
   }
   
   private val members = new ArrayBuffer[Member]
   
   def join(name:String) = {
        val m = new Member(name)
        members += m
        m
   }
}
```
//不同的NetWork实例的Member不能互相复制，因为不是相同的Member类。
解决方式
1. 伴生对象中定义
```scala
object Network{
    class Member(val name:String){
        val contacts = new ArrayBuffer[Member]
    }
}
class NetWork{
      ...
}
```

2. 类型投影
```scala
class Network{
    class Member(val name:String){
        val contacts = new ArrayBuffer[Network#Member]
    }
}
```


### 第六章 对象

单例对象

scala没有静态方法和静态字段，你可以用object这个语法结构来达到同样的目的。
对象定义了某个类的单个实例，包含了你想要的特性。对象的构造器在改对象第一次使用的时候被调用。
>对象本质上可以拥有类的所有特性----它甚至可以扩展其他类或特质，只有一个例外：不能提供构造器参数。

伴生对象

类和它的伴生对象可以相互访问私有特性。它们必须存在同一个源文件中。

扩展类或特质的对象

一个object可以扩展类以及一个或多个特质，其结果是一个扩展了指定类以及特质的类的对象，同时拥有在对象定义中给出的所有特性。

apply方法
```scala
//调用的是apply(100),输出一个单元素（整数100）的Array[Int];
Array(100) 
//调用构造器this(100),结果是Array[Nothing],包含了100个null元素
new Array(100)
```

### 第七章 包和引入

同一个包可以定义在多个文件中。你也可以在同一个文件当中为多个包贡献内容
>源文件的目录和包之间并没有强制的关联关系。你不需要将Employye.scala和Manager.scala放在com/horstmann/impatient目录当中。

```scala
package  com {
    package horstmann{
        package impatient {
            class Employee
              ...
        }
    }
}

package org{
    package bigjava{
        class Counter
          ...
    }
}
```

文件顶部标记法

除了我们到目前为止看到的嵌套标记法外，你也可以在文件顶部使用package语句，不带花括号。例如：

```scala
package com.horstmann.impatient
package people
class  Person
...
```
等同于
```scala
package com.horsmann.impatient{
    package people{
        class Person
          ...
    }
}
```

任何地方都可以声明引入

在Scala中，import语句可以出现在任何地方，并不仅限于文件顶部。import语句的效果一直眼神到包含该语句的块末尾。例如：
```scala
class Manager{
    jimport scala.collection.mutable._
    val subordinates = new ArrayBuffer[Emplyee]
}
```
重命名和隐藏方法
```scala
//部分引用 选取器
import java.awt.{Color,Font}
//重命名
import java.util.{HashMap => JavaHashMap}
```

隐式引用
```scala
//这种引用被允许可以覆盖之前的引用
import java.lang._
import scala._
import Predef._
```

### 第八章 继承

在Scala中重写一个非抽象方法必须使用override修饰符

类型检查和转换
要判断某个对象是否属于某个给定的类，可以用isInstanceOf方法。如果测试成功，你就可以用asInstanceOf方法将引用转换为子类的引用：
```scala
if (p.isInstanceOf[Employye]){
    val s = p.asInstanceOf[Employee]
      ...
}
```

如果相判断p指向的是一个Employee对象 但又不是其子类的话，可以用：
```scala
if (p.getClass == classOf[Employee])
```

> 子类的辅助构造器永远都不可能直接调用超类的构造器，子类的辅助构造器最终都会调用主构造器。
只有主构造器可以调用超类的构造器。

```scala
//scala写法
class Employee(name:String,age:Int,val salary:Double) extends Person(name,age)
//等同于java写法
public class Employee extends Person {
    private double salary;
    public Employee(String name,int age,double salary){
        super(name,age);
        this.salary = salary;
    }
}
```

重写字段
- def只能重写另一个def
- val只能重写另一个val或不带参数的def
- var只能重写另一个抽象的var


Scala继承层级

基本类型以及Unit类型扩展至AnyVal,所有其他类都是AnyRef的子类, AnyVal和AnyRef都是扩展自Any类，Any类是整个继承层级的根节点。
AnyRef追加了来自Object类的监视方法wait和notify/notifyAll同时提供一个带函数参数的方法synchronized。 

继承级另外一端是Nothin和Null类型。
Null类型的唯一实例是null值。你可以将null赋值给任何引用，单不能赋值给值类型。举例来说，我们不能讲Int设为null。这比Java更好，在Java中我们可以将Integer包装类引用设为null.
Nothing类型没有实例。它对泛型结构时常有用。


### 第九章 文件和正则表达式
空 都易于理解

### 第十章 特质

当做接口使用的特质

你不需要将方法声明为abstract特质中未被实现的方法默认就是抽象的。
和Java一样，Scala只能有一个超类，但可以有任意数量的特质。
```scala
trait Logger {
    def log(msg:String)
}
class ConsoleLogger extends Logger{ 
    //不需要写override
    def log(msg: String){
        println(msg)
    }
}

//如果特质混入另一个特质，其中抽象方法log,需要调用super.log 则需要标记。
trait TimestampLogger extends Logger {
    abstract override def log(msg: String){
        super.log(new java.util.Date() + "" + msg)
    }
}


```

带有具体实现的特质
>让特质拥有具体的方法存在一个弊端，当特质改变是，所有混入了改特质的类都必须重新编译。


带有特质的对象
```scala
trait Logged{
    //log 什么也没做
    def log(msg:String){
    
    }
}

//现在 SavingsAccount 什么都不会被记录到日志中，看上去似乎毫无意义
//但是你可以在构造具体对象的时候“混入”一个更好的日志记录器的实现。
class SavingsAccount extends Account with Logged{
    def withdraw(amount:Double){
        if(amount > balance) log("Insufficient funds")
        else   ...
    }
}

trait ConsoleLogger extends Logged{
    override def log(msg :String){
        println(msg)
    }
}

//你可以再构造对象的时候加入特质
val acct = new SavingsAccount with ConsoleLogger
//另一个对象可以加入不同的特质
val acct2 = new SavingsAccount with FileLogger
```

叠加在一起的特质

特质上，super调用的是特质层级中的下一个特质，具体是哪一个，要根据特质添加的顺序来决定。
一般来说，特质从最后一个开始被处理。

>对特质而言，你无法从源码判断super.someMethod会执行哪里的方法。确切的方法依赖于使用这些特质的对象或类给出的顺序。
这使得super相比在传统的继承关系中要灵活得多。

>如果你需要控制具体是哪一个特质的方法被调用，则可以在方括号中给出名称: super[ConsoleLogger].log(...)。
这里给出的类型必须是直接超类型；你无法使用继承层级中更远的特质或类。


特质中具体字段

特质中的字段可以是具体的，也可以是抽象的。如果你给出了初始值，那么字段就是具体的。
通常对于特质中的每一个具体字段，使用该特质的类都会获得一个字段与之对应。
这些字段不是被继承的；它们只是简单地加到了子类当中。
看上去是个席位的差别，单这个区别很重要。
```scala
trait ShortLogger extends Logged{
    val maxLength = 15 //具体字段
}

class Account {
    var balance = 0.0
}

class SavingsAccount extends Account with ConsoleLogger with ShortLogger {
    var interest = 0.0
    def withdraw(amount:Double){
        if(amount >balance) log("Insufficeent funds")
        else   ...
    }
}

//其中子类字段 interest , maxLength
// 超类字段 balance
```
在JVM中，一个类智能扩展一个超类，因此来自特质的字段不能以相同的方式继承。
由于这个限制，maxLength被直接加到了SavingsAccount类中，跟interest字段排在一起。
你可以把具体的特质字段当做是针对使用该特质的类的“装配指令”。
任何通过这种方式被混入的字段都自动成为该类自己的字段。

特质中的抽象字段

>特质中的未被初始化的字段在具体的子类中必须被重写。

特质的构造顺序

- 首先调用超类的构造器
- 特质构造器在超类构造器之后，类构造器之前执行。
- 特质由左到右被构造。
- 每个特质当中，父特质先被构造。
- 如果多个特质公有一个父特质，而那个父特质已经被构造，则不会被再次构造。
- 所有特质构造完毕，子类被构造。

```scala
class SavingsAccount extends Account with FileLogger with ShortLogger
```
构造顺序
1. Account(超类)
2. Logger(第一个特质的父特质)
3. FileLogger (第一个特质)
4. ShortLogger(第二个特质)。注意它的父特质Logger已被构造
5. SavingsAccount(类)

构造器的顺序是类的线性化的反向。
```scala
如果C extends C1 with C2 with ...with Cn 则 linc(C) = C>> lin(Cn) >> ...>>lin(C2) >> lin(C1)
```

这里 >> 意思是 串联并去掉重复项，右侧胜出。

```scala
lin(SavingsAccount) = SavingsAccount >> lin(ShortLogger) >> lin(FileLogger) >> lin(Account)
                    = SavingsAccount >> (ShortLogger >> Logger) >> (FileLogger >> Logger) >> lin(Account)
                    = SavingsAccount >> ShortLogger >> FileLogger >> Logger >> Account
                    (省略了线性化末端类型：ScalaObject、AnyRef、Any)

```
                    
                    
初始化特质中的字段
>特质不能有构造器参数。每个特质都有一个无参数的构造器，缺少构造器参数是特质与类之间唯一的技术差别。


扩展类的特质
特质也可以扩展类，这个类将会自动成为所有混入该特质的超类
```scala
trait LoggedException extends Exception with Logged{
    def log(){
        log(getMessage())
    }
}
```
如果一个类已经扩展了另一个类同时混入其他特质，扩展的类的超类必须是混入的特质的超类的子类。 否则不可以混入特质。

自身类型
```scala
// 没有扩展Exception类，但是有个自身类型Exception. 意味着，它只能混入Excetpion的子类。
trait LoggedException extends Logged {
    this: Exception => def log() {log(getMessage())}
}
```
带有自身类型的特质和带有超类型的特质很相似。两种情况都能确保混入该特质的类能够使用某个特定类型的特性。

```scala
//自身类型也同样可以处理结构类型（structural type） 这种类型只给出类必须拥有的方法，而不是类的名称。
trait LoggedException extends Logged{
    this: {
        def getMessage(): String
    } => def log(){
        log(getMessage())
    }
}
```

### 第十一章 操作符 
空 都易于理解

### 第十二章 高阶函数 

高阶函数： 接收函数参数的函数。

闭包

在Scala中，你可以再任何地方作用域内定义函数：包，类甚至是另一个函数或方法。
在函数体内，你可以访问到相应作用域内的任何变量。你的函数可以在变量不再处于作用域内时被调用。
```scala
def mulBy(factor:Double) = (x:Double) => factor * x
val triple = mulBy(3)
val half = mulBy(0.5)
//单一你 42  7 
//每一个返回的函数都有自己的factor设置
println(triple(14) + "" + half(14))
```

柯里化

柯里化（currying,以逻辑学家Haskell Brooks Curry的名字命名）指的是将原来接收两个参数的函数变成新的接收一个参数的函数过程。
新的函数返回一个以原有第二个参数作为参数的函数。


