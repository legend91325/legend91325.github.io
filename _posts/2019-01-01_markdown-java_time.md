---
title: "读懂时间故事"
layout: post
date: 2019-01-01 00:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Java
- history note
category: blog
author: wukong
description: time
---
>以前博客文章，时间点无法追溯，统一放在 2019-01-01 00:00:00。

题外话，祝各位程序猿们中秋佳节快乐~~~O(∩_∩)O~。
##追本溯源
###历史

####GMT
百度百科：[格林威治时间](http://baike.baidu.com/view/856.htm)
####UTC
百度百科：[协调世界时](http://baike.baidu.com/view/42936.htm?fromtitle=UTC&fromid=5899996&type=syn)

两者比较：
>**GMT**：最初确立的世界标准时间，名字由来是因为英国的皇家格林尼治天文台而得名，因为本初子午线被定义在通过那里的经线。自1924年2月5日开始，格林尼治天文台每隔一小时会向全世界发放调时信息。由于地球每天的自转是有些不规则的，而且正在缓慢减速，因此，格林尼治时间已经不再被作为标准时间使用。新的标准时间，是由原子钟报时的协调世界时（UTC）。**--百度百科**

>**UTC**：是目前使用的世界标准时间，其以原子时秒长为基础，在时刻上尽量接近于格林尼治平时。国际原子时的误差为每日数纳秒，而世界时的误差为每日数毫秒。对于这种情况，一种称为协调世界时的折衷时标于1972年面世。为确保协调世界时与世界时相差不会超过0.9秒，在有需要的情况下会在协调世界时内加上正或负闰秒。因此协调世界时与国际原子时之间会出现若干整数秒的差别。位于巴黎的国际地球自转事务中央局负责决定何时加入闰秒。 **--维基百科**

这里要提下有趣的事情是，因为UTC闰秒的缘故，java中的秒可能为60或61，但java文档中有说这个是依据计算机的环境，无法保证准确性囧。
根据这两个比较，GMT和UTC在正常生活中可以指同一个时间，除非你做的东西对时间的**准确性**极为敏感。


###一些重要概念：
####时区
1884年国际经线会议规定，全球按经度分为24个时区，每区各占经度15°。
以本初子午线为中央经线的时区为零时区，由零时区向东、西各分12区，东、西12区都是半时区，共同使用180°经线的地方时。


####CST ：China Standard Time UTC+8:00 中国标准时间(北京时间)，在东八区
俺们这块的时区就是 UTC +8 ,相当于我们国家的使用的时间比世界协调时间快了8小时。

####unix时间戳
是从1970年1月1日（UTC/GMT的午夜）开始所经过的秒数，不考虑闰秒。
这里在放一个链接，关于为什么都要从1970年算起，以前也真是懒的思考这个问题，拿来就用，以后真的需要做什么事情都先问个为什么了。O(∩_∩)O哈哈~
[计算机时间、unix时间、linux时间、java时间为何以1970年1月1日为原点？从1970年1月1日开始计算？](http://blog.csdn.net/nishuishenhan/article/details/6020873)
百度百科：[unix时间戳](http://baike.baidu.com/view/821460.htm)

##java中使用
###时间类 timeStamp date Calendar
```java

    long millisTime = System.currentTimeMillis();
	Timestamp timestamp = new Timestamp(millisTime);
	Time time = new Time(millisTime);
	Date date = new Date(millisTime);
	Calendar calendar = Calendar.getInstance();
	
	System.out.println(time.getTime());
	System.out.println(date.getTime());
	System.out.println(timestamp.getTime());
	System.out.println(calendar.getTimeInMillis());
```
以上构建之后，输出的都是同样的值，因为前三个是返回是距离**January 1, 1970, 00:00:00 GMT**的毫秒，而`calendar`返回的是**the current time as UTC milliseconds from the epoch**。上面说了GMT和UTC可以看成指同一个时间。所以值就是一样的了。一般网络通信传输时间都是先转成毫秒之后再转回来，因为这里面不会包含**时区**，这样才是准确的。

在java程序中一般都会使用`Calendar`因为其他几个类里面的方法几乎都不建议使用了，`Calendar`中提供了丰富的api，各位程序猿们使用的时候可以自行查找。
```java
    Calendar calendar = Calendar.getInstance();
    //获取所有时区信息
	TimeZone.getAvailableIDs();
	//设置一个时区
	calendar.setTimeZone(TimeZone.getTimeZone("GMT+13"));
	//打印该时区目前小时时间，24制
	System.out.println(calendar.get(calendar.HOUR_OF_DAY));
```

###格式化时间显示类SimpleDateFormat

顺带再提下Java的`SimpleDateFormat`类，用来把时间格式化显示的，显示的样式很丰富，看下列表格列举的。

|Letter | Date or Time Component|  Presentation   | Examples|
|-----  |-----                  |----             |---------|
|G      |Era designator         |  Text           |      AD |
|y      | Year                  |  Year           | 1996; 96|
|M      |Month in year          |  Month          | July; Jul; 07|
|w      |Week in year           |  Number         | 27
|W      |Week in month          | Number          | 2
|D      |Day in year            | Number          | 189
|d      |Day in month           | Number          | 10
|F      |Day of week in month   |  Number         | 2
|E      |Day in week            | Text            | Tuesday; Tue
|a      |Am/pm marker           | Text            | PM
|H      |Hour in day (0-23)     | Number          | 0
|k      |Hour in day (1-24)     |Number           | 24
|K      |Hour in am/pm (0-11)   |Number           | 0
|h      |Hour in am/pm (1-12)   |Number           | 12
|m      |Minute in hour         |Number           | 30
|s      |Second in minute       |Number           | 55
|S      |Millisecond            |Number           | 978
|z      |Time zone              |General time zone| Pacific Standard Time; PST; GMT-08:00
|Z      |Time zone              |RFC 822 time zone| -0800 

下面这个方法是我在这里[How to convert a String to a Date using SimpleDateFormat?](http://stackoverflow.com/questions/9872419/how-to-convert-a-string-to-a-date-using-simpledateformat)看到的。
相比惯例的格式化时间方法，是把每种格式类型都定义好之后再用分别的方法调用，这个写成了一个统一的方法，虽然效率上没有改进，但是这种写法还是对可阅读性有了不少的提高啊。
```java

/**
     * Format a time from a given format to given target format
     * 
     * @param inputFormat
     * @param inputTimeStamp
     * @param outputFormat
     * @return
     * @throws ParseException
     */
    private static String TimeStampConverter(final String inputFormat,
            String inputTimeStamp, final String outputFormat)
            throws ParseException {
        return new SimpleDateFormat(outputFormat).format(new SimpleDateFormat(
                inputFormat).parse(inputTimeStamp));
    }
```


##头痛的mysql中时间的使用

###各有千秋
我们来一一介绍下mysql中的时间，
**DATE**
显示的格式为`YYYY-MM-DD` 范围从 '1000-01-01' 到 '9999-12-31' 占用3个字节

**DATETIME**
显示的格式为`YYYY-MM-DD HH:MM:SS` 范围从'1000-01-01 00:00:00' 到'9999-12-31 23:59:59' 占用8个字节

**TIMESTAMP**
显示的格式`YYYY-MM-DD HH:MM:SS` 范围从'1970-01-01 00:00:01'到 '2038-01-19 03:14:07' 占用4个字节 相信各位如果读了上面unix时间戳的来历，应该明白为什么占用4个字节。

区别：
1. 首先要提的就是`datetime`和`timestamp`之间最大的不同就是在于，timestamp随着时区的变化而变化，默认会根据系统的时区设置来存储时间，打个比方，如果你在东八区存储一个时间，之后把数据库系统设置另一个时区则取出来的值发生变化。如果你存储数据的库分布在不同的时区时你就要考虑这种情况，而采用`datetime`,因为它是不受时区影响的。
2. 三种时间格式的非法值都会变成'0000-00-00' 或 '0000-00-00 00:00:00'
3. `timestamp`提供了每次数据更新时，自动更新为现有的时间功能，这个功能比较有用。
4. `date`会自动处理一些格式比如'10:11:12'会理解成'2010-11-12'，年值在'00-69'mysql会认为是'2000-2269',年值在'70-99'mysql会认为'1970-1999'话说mysql还算智能哈。

参考文章：
[The DATE, DATETIME, and TIMESTAMP Types](https://dev.mysql.com/doc/refman/5.7/en/datetime.html)
[java编程中遇到的时区与时间问题总结](http://www.rigongyizu.com/java-timezone-time-issue-summary/)
[How to convert a String to a Date using SimpleDateFormat?](http://stackoverflow.com/questions/9872419/how-to-convert-a-string-to-a-date-using-simpledateformat)
[计算机时间、unix时间、linux时间、java时间为何以1970年1月1日为原点？从1970年1月1日开始计算？](http://blog.csdn.net/nishuishenhan/article/details/6020873)
[协调世界时](http://baike.baidu.com/view/42936.htm?fromtitle=UTC&fromid=5899996&type=syn)
[格林威治时间](http://baike.baidu.com/view/856.htm)
[unix时间戳](http://baike.baidu.com/view/821460.htm)
