---
title: "Java 程序性能优化"
layout: post
date: 2016-04-08 21:48
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Java
- optimize
star: true
category: blog
author: wukong
description: Java performance optimize
---

## Java设计优化

#### 1. 单例模式

> 静态内部类，调用时才初始化实例，即可以做到延迟加载，也不必使用同步关键字
```java
public class StaticSingleton{
    private StaticSingleton(){}
    
    private static class SingletonHolder{
        private static SingletonHolder instance = new SingletonHolder();
    }
    public static StaticSingleton getInstance(){
        return SingletonHolder.instance;
    }
}
```
> 序列化，反序列化 通过实现私有的readReslove()方法，替换原本的返回值，从而实现单例。

#### 2. 动态代理
> 动态代理是指在运行时，动态生成代理类。即，代理类的字节码将在运行时生成并载入当前ClassLoader。
性能
ASM > Javassist, CGLB > JDK
复杂程度
ASM > Javassist,CGLB,JDK

#### 3. 装饰者模式
> 合成/聚合复用原则，代码复用应尽可能使用委托，而不是使用继承。通过委托机制，
复用系统中的各个组件，在运行时，可以将这些我刚给你呢组件进行叠加，从而构造一个“超级对象”
，使其拥有所有这些组件的功能。
```java
//组件接口
public interface IPacketCreator{
    String handleContent();
}
//被装饰者
public class PacketBodyCreator implements IPacketCreator{
    
    @Override
    public String handleContent(){
        return "Content of Packet";
    }
}

//装饰者，维护核心组件component
public abstract class PacketDecorator implements IpacketCreator{
    IPacketCreator component;
    public PacketDecorator(IPacketCreator c){
        component = c;
    }
}
```
> 通过层层构造和组装这些装饰者和被装饰者到一个对象中，使其有机结合在一起工作。
Java中的OutputStream,InputStream为核心的装饰者模式实现。
```
public static void main(String[] args) throws IOException{
    DataOutputStream out = new DataOutputStream(new BufferedOutputStream(new FileOutputStream("")))
}
```

#### 时间换空间 、空间换时间
> 时间换空间
```
a = a+b;
b = a-b;
a = a-b;
```
> 空间换时间
```
//下标索引 用空间排序 时间复杂度O(n)
public static void spaceTotime(int[] array){
    int i = 0;
    int max = array[0];
    int length = array.length;
    for(int i=1;i<length;i++){
       if(array[i]>max){
           max = array[i]
       }    
    }
    //根据最大值分配长度
    int[] temp = new int[max+1]
    for(int i=0;i<length;i++){
        temp[array[i]] += 1; 
    }
    for(int j=0;j<max+1;j++){
        if(temp[j]>0){
            for(int m=0;m<temp[j];m++){
                System.out.println(j)
             }
        }
    }
}

```

## Java程序优化

> subString() 方法的内存泄漏[旧版本的问题，JDK源码发现已经改进]
内部实现，为了搞笑且快速地共享String内的char数组对象，通过偏移量截取字符串来实现，问题是，String原
生内容也会被复制到子串中，如果原始字符串很大，截取字符串很小，这样会浪费很多内存空间。
是一种空间换时间的策略。
使用方式要是
```java
//这样就不会造成内存泄漏
new String(str.substring(begin,end))
```

> 简单分割 不用String.split() 而是使用StringTokenizer,或indexOf()自己实现分割。
> StringBuilder,StringBuffer显示使用以及手动指定容量都可以提升性能

## 核心数据结构
>LinkedHashMap 维护了元素次序，提供两种顺序，一个是元素插入顺序 accessOrder 为False, 一个是最近访问顺序 accessOrder 为True

>随机访问 判断是否实现了RandomAccess接口，基于数组的List都实现了，基于链表的没有实现。

## 引用类型

> 强引用
- 可以直接访问目标对象
- 所执行的对象在任何时候都比被系统回收，JVM宁愿抛出OOM异常
- 因此可能会导致内存泄漏

> 软引用
- 通过SoftReference来使用。
- JVM会根据当前堆得使用情况来判断核实回收，当堆使用率临近阈值，才会去回收软引用对象。

> 弱引用
- 通过WeakReference来使用
- 系统GC时，只要发现弱引用，就会将对象进行回收

> 软引用和弱引用都适合保存可有可无的缓存数据，如果内存不足时，缓存数据会被回收，不会导致内存溢出。

> 虚引用
- PhantomReference
- 虚引用的对象和没有引用几乎是一样的，随时都可能被垃圾回收器回收。虚引用必须和引用队列ReferenceQueue一起使用，
它的作用在于跟踪垃圾回收过程。

## 有助于改善性能的技巧
> 慎用异常，try-catch语句对系统性能而言是非常糟糕。尽量不要把try-catch应用于循环之中。
> 位运算
> 替换switch
> 一维数组代替二维数组
> 提取表达式 ： x=d*a*b/3*4*a; y=e*a*b/3*4*a -->t=a*b/3*4*1; x=d*t;y=e*t;
> 展开循环
```java
for(int i=0;i<99999;i++){
    array[i]=i;
}
//替换
for(int i=0;i<99999;i+3){
    array[i]=i;
    array[i+1]=i+1;
    array[i+2]=i+2;
}
```