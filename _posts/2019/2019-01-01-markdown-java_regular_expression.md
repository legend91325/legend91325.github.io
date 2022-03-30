---
title: "java正则掉坑实录"
layout: post
date: 2019-01-01 00:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Java
- history note
category: blog
author: WuKongCoder
description: time
---
>以前博客文章，时间点无法追溯，统一放在 2019-01-01 00:00:00。


大学时候对正则很感兴趣，还特意学习了perl的《精通正则表达式》，平时开发中也会经常用到，但是都写的比较简单的正则，也没有太考虑效率，额是主Java语言的。

最近正好写了个抓取代理ip地址的程序，用到了正则匹配，本来挺简单的一个问题，但是对正则的贪婪和非贪婪匹配没有理解深刻，导致在上面耗了很长时间。

先简单说下贪婪和非贪婪匹配：
```java
String regexGreedy = "<.*>"
String regexNoGreedy = "<.*?>"

String content = "<tr><td>abc</td><tr>"
```
上面第一个`regexGreedy`正则匹配到的是`content`全部内容，而第二个`regexNoGreedy`匹配到的是`<tr>`内容。
显而易见，一种是**要最多**，另一种是**要最少**。

好了，其实问题很简单，但是实战中就有可能出现问题，以下是我的实战内容：

需求：我想抓取某个公布代理ip地址的网站上数据。
程序：
```java
public class Proxy {
	
	private final static ConcurrentHashMap<String, Integer> ipMap = new ConcurrentHashMap<String, Integer>();
	
	public static Logger logger = Logger.getLogger(Proxy.class);
	
	//地址就不放上来了。。。
	private static String  proxyUrl = "xxx";
	
	private static Header header_Encode = new Header("Accept-Encoding","Accept-Encoding:gzip,deflate,sdch");

	/**
	 * 刷新代理ip
	 */
	public static void flush(){
		
		org.apache.commons.httpclient.HttpClient httpClient = new org.apache.commons.httpclient.HttpClient();
		GetMethod getMethod = new GetMethod(proxyUrl);
		getMethod.setRequestHeader(header_Encode);
		InputStream urlStream =null;
		BufferedReader reader = null;
		try {
			httpClient.executeMethod(getMethod);
			if(getMethod.getStatusLine().getStatusCode()==HttpStatus.SC_OK){
				System.out.println(getMethod.getRequestHeader("Accept-Encoding").getValue());
				try{
					urlStream = new GZIPInputStream(getMethod.getResponseBodyAsStream());
				}catch(IOException e){
					urlStream =  new BufferedInputStream(getMethod.getResponseBodyAsStream());
				}
				  
                reader = new BufferedReader(new InputStreamReader(urlStream,"utf-8"));  
                StringBuilder content = new StringBuilder();  
                String line = null;
                while((line = reader.readLine()) != null) {  
                	content.append(line);
                }
                //System.out.println(content.toString());
                parser(content.toString());
                  
			}
		} catch (HttpException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}finally{
			
			try {
				
				if(reader!=null){
					reader.close();
				}
				if(urlStream!=null){
					urlStream.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
            
		}
		
	}
	
	
	private static HashMap<String,Integer> parser(String content) {
		HashMap <String ,Integer> map = new HashMap<String,Integer> ();
		
		String regexBody = "<tbody>(.*)</tbody>";
		String regexTr ="<tr><td>([^<]*)</td><td>([0-9]*)</td>.*</tr>";
		Pattern pattern = Pattern.compile(regexBody);
		Matcher matcher = pattern.matcher(content);
		
		if(matcher.find()){
			content = matcher.group(1);
		}
		
		content = content.replaceAll("[\\s*\\n*\\s*]", "");
		System.out.println(content);
		pattern = Pattern.compile(regexTr,Pattern.MULTILINE | Pattern.DOTALL);
		matcher = pattern.matcher(content);
		
		String ip = "";
		Integer port = null;
		while(matcher.find()){
			
			try{
				ip = matcher.group(1);
				port = Integer.parseInt(matcher.group(2));
				map.put(ip, port);
				System.out.println("ip="+ip+"|port="+port);
			}catch(Exception e){
				logger.error(e.getMessage(),e);
			}
		}
		
		return map;

	}
	
	public  static void main(String[] args){
		Proxy.flush();
	}

}
```
程序比较简单，看一下大致都懂，但是我当时测试的时候无论怎么抓取都是抓取到网页上最后一组数据。
最终定位的问题就在正则匹配上:
`String regexTr ="<tr><td>([^<]*)</td><td>([0-9]*)</td>.*</tr>";`
这个正则是贪婪匹配，所以是**要最多**。

修改后
`String regexTr ="<tr><td>([^<]*?)</td><td>([0-9]*?)</td>.*?</tr>";`
为非贪婪匹配，就可以捕获到所有了。

问题很简单，贪婪和非贪婪概念我也都懂，但是要灵活的在实战中运用就需要多多使用正则，积累经验，争取下次不要再掉坑了。
