---
title: "Scala functions vs Methods"
layout: post
date: 2015-03-05 17:25
image: /assets/images/markdown.jpg
headerImage: false
tag:
- scala
- history note
category: blog
author: wukong
description: Scala functions vs methods
---

# Scala Functions vs Methods

标签（空格分隔）： scala

---
写在开头，今年不经意间接触到了scala语言，以前一直在使用java语言，现在对scala比较有兴趣，最近用业余时间在学习这方面知识，已经看完《快学scala》正在看《scala编程》，这边文章是我在其他个人博客中看到的，感觉对我理解scala中函数式编程很有帮助，所以想翻译过来,让自己对这内容理解更透彻些。

**附上博客原文地址**：[Scala Functions vs Methods](http://jim-mcbeath.blogspot.com/2009/05/scala-functions-vs-methods.html)
以下是本文的翻译，不足之处还望指出帮助改正：

scala中既有函数也有方法。大多数时候，我们忽略了两者之间的不同，但是有时我们必须面对它们不是同样一件事这个事实。

在我的scala基本语法中我有提过在讨论中我使用方法和函数互相替换使用。那是简单化的。在大多数场景下，你可以忽略函数和方法两者之间的区别，仅仅把他们当做同一件事就可以，但是偶尔你可能会在某个场景下遇到他们不同之处。与我们对待质量和重量的看法十分相似。在我们日常在地球表面生活，我们把两者当做可以互换的单元，1Kg与2.2磅相同。但是它们也不是完全一样，当宇航员在月球的表面行走，他的质量（千克）没有改变，但是他的重量(磅)已经变成在地球的六分之一了。

在千克与磅的对比中，相比你在月球表面走动，你更有可能遇到在某个场景下区分scala中函数和方法不同之处。所以什么时候忽略两者之间的不同，什么时候你又需要注意两者之间的不同呢？一旦你理解这不同之处你就可以回答这个问题了。

一个scala 方法，在java中是类的一部分，它有一个名字，一个签名，可选的注解 和字节码。

在scala中一个函数是一个完整的object，在scala中有一系列的特质代表各种各样的带参数的函数：Function0, Function1, Function2等等。 作为一个混入了这些特质之一的类的实例化，一个函数object有许多方法。这些方法之一就是apply方法，这个方法包含了实现这个函数的代码。 scala中apply有特殊的语法：如果你写一个函数名紧接着是一个带一系列参数列表的括号（或者仅有不带参数的括号），scala会将调用转换成对应名的object的apply方法。当我们创建一个变量，值是一个函数的object之后我们引用这个带着括号的变量，它将会自动转成这个函数object的apply方法的调用（译者注：看过scala语法的应该知道，就是隐式的调用了apply方法，这里翻译的有点拗口）。

 
当我们把一个方法当做一个函数，比如我们把它赋值给一个变量，scala确实会创建一个函数object，其中的apply方法调用原始的方法，并且是将这个object赋值给这个变量。定义一个函数的对象，把它赋值给一个变量，因为需要定义一个功能上与原始方法等价的函数，并实例化这个函数对象，这种方式的开销会更加消耗内存。因此，你不会想让每一个方法都变成一个函数；但是函数会赋予你更加强大的功能，这功能不仅仅是方法 同时在某些情况下所提供的功能值的我们使用更多的内存（译者注：丫的，不直接说明白具体什么地方。哈哈，其实在后面）。


让我们看看这个机制的详细细节。创建一个test.scala文件，内容如下：
```scala
class test {
    def m1(x:Int) = x+3
    val f1 = (x:Int) => x+3
}
```

用scalac编译这个文件，得到编译后的文件，scala创建了两个文件：test.class 和 test$$anonfun$1.class。这个多余的奇怪文件是函数对象的匿名类，是scala创建用来对应赋值给f1的函数表达式函数。如果在你的测试类中使用更多的函数表达式，同样将会有更多的匿名类产生，即使你再一次将同样的函数表达式写了一遍。

如果你用`javap`运行这个测试类，你将会看到：
```scala
Compiled from "test.scala"
public class test extends java.lang.Object implements scala.ScalaObject{
    public test();
    public scala.Function1 f1();
    public int m1(int);
    public int $tag()       throws java.rmi.RemoteException;
}
```

使用`javap`运行`test$$anonfun$1`函数类，产出：
```scala
Compiled from "test.scala"
public final class test$$anonfun$1 extends java.lang.Object implements scala.Function1,scala.ScalaObject{
    public test$$anonfun$1(test);
    public final java.lang.Object apply(java.lang.Object);
    public final int apply(int);
    public int $tag()       throws java.rmi.RemoteException;
    public scala.Function1 andThen(scala.Function1);
    public scala.Function1 compose(scala.Function1);
    public java.lang.String toString();
}
```
 
这个类实现了Function1的接口，我们知道那是带一个参数的函数。你可以看到这个类中包含一把方法，包含apply方法。


你也可以通过使用一个存在的方法来定义一个函数，引用函数名 接着一个空格和一个下划线。修改test.scala 加上另一行：
```scala
class test {
    def m1(x:Int) = x+3
    val f1 = (x:Int) => x+3
    val f2 = m1 _
}
```

这`m1 _`语法 告诉scala 把`m1`当做一个函数而不是通过调用那个方法来使用产生的值。作为另一种选择，你可以详细的声明 类型`f2`,这样你不需要包含一个末尾下划线：
```scala
 val f2 : (Int) => Int = m1
```

一般情况下，如果scala期望一个函数类型，你可以传递一个方法名，让它自动转换成一个函数。例如，如果你调用一个接收一个函数作为它的参数的方法，你可以通过恰当的方法签名来提供这个参数，而不用包含末尾下划线。

回到我们的测试文件上来，现在当你编译这个`test.scala`，将会有两个匿名的类，一个是`f1`类一个是`f2`类。你可以使用两个方式任意一个来定义`f2`,都会产生一个唯一的类文件。

如果你使用`javap`时 加上选项`-c`，你可以得到第二个匿名类的源码，你可以看到测试类中的`apply`方法调用了`m1`的方法。
```scala
public final int apply(int);
  Code:
   0:   aload_0
   1:   getfield        #17; //Field $outer:Ltest;
   4:   astore_2
   5:   aload_0
   6:   getfield        #17; //Field $outer:Ltest;
   9:   iload_1
   10:  invokevirtual   #51; //Method test.m1:(I)I
   13:  ireturn
```

让我们瞄准scala的编译器，看看它是如何工作的。在接下来的例子中，加粗的黑体表示输入的文字，紧接着标注的是输出的文字。

scala> **def m1(x:Int) = x+3**
m1: (Int)Int

scala> **val f1 = (x:Int) => x+3**
f1: (Int) => Int = <function>

scala> **val f2 = m1 _**
f2: (Int) => Int = <function>

scala> **m1(2)**
res0: Int = 5

scala> **f1(2)**
res1: Int = 5

scala> **f2(2)**
res2: Int = 5

注意这`m1`和`f1`的签名是不同的，`(Int)Int`对于一个方法签名来说代表接受一个Int类型的参数，返回一个Int类型的值，这`(Int) =>Int`签名意味着一个接受一个Int参数的函数，返回一个Int值。

在这点上，我们似乎有一个方法`m1`和两个函数`f1`和`f2`，他们的功能都是一样的。但是`f1`和`f2`确实是两个变量，他们是由一个是实现了`Function1`接口的生成类生成的实例。实例对象有很多`m1`没有的方法。
```scala
scala> f1.toString
res3: java.lang.String = <function>

scala> (f1 andThen f2)(2)
res4: Int = 8
```

因为`m1`本身就是方法，不同于`f1`,你不能在它上面调用方法：
```scala
scala> m1.toString
<console>:6: error: missing arguments for method m1 in object $iw;
follow this method with `_' if you want to treat it as a partially applied function
       m1.toString
       ^
```

注意，每一次你引用一个方法作为一个函数，scala都会创建一个单独的对象。
```scala
scala> val f3 = m1 _
f3: (Int) => Int = <function>

scala> f2 == f3
res6: Boolean = false
```

即使`f2`和`f3`都引用了`m1`，两者做的事情是一样的，但是在scala中它们不认为是相等的，因为默认继承的比较方法是通过唯一标识去比较的，这是两个不同的对象。如果你想让两个函数值相等，你需要确保引用它们引用相同的函数对象。
```scala
scala> val f4 = f2
f4: (Int) => Int = <function>

scala> f2 == f4
res7: Boolean = true
```

这有几个例子来表明表达式`m1 _`事实上是一个函数对象：
```scala
scala> m1 _
res8: (Int) => Int = <function>

scala> (m1 _).toString
res9: java.lang.String = <function>

scala> (m1 _).apply(3)
res10: Int = 6
```

scala 2.8.0版本中，表达式`(m1 _)(3)`也会返回相同的值(以前的版本中有一个bug,会引起类型没匹配的错误)


方法与函数之间还有一些其他的不同之处，方法可以是一个类型参数化，但是一个匿名的函数则无法做到：
```scala
scala> def m2[T](x:T) = x.toString.substring(0,4)
m2: [T](T)java.lang.String

scala> m2("abcdefg")
res11: java.lang.String = abcd

scala> m2(1234567)
res12: java.lang.String = 1234
```

但是，如果你愿意将你的函数定义为一个明确的类的话，你同样可以类型参数化它：
```scala
scala> class myfunc[T] extends Function1[T,String] {
     |     def apply(x:T) = x.toString.substring(0,4)
     | }
defined class myfunc

scala> val f5 = new myfunc[String]
f5: myfunc[String] = <function>

scala> f5("abcdefg")
res13: java.lang.String = abcd

scala> val f6 = new myfunc[Int]
f6: myfunc[Int] = <function>

scala> f6(1234567)
res14: java.lang.String = 1234
```

所以让我们继续，通过将除以2.2，将磅转换成千克（除非你是一个宇航员），但是当你在scala中混用函数和方法时，一定要记住他们不是在做同一件事。
