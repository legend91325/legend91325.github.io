---
title: "Java 程序性能优化"
layout: post
date: 2016-04-08 21:48
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Java
- optimize
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

