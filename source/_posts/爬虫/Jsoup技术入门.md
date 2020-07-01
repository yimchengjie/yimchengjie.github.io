---
title: Jsoup技术入门
categories:
  - 爬虫
tags:
  - 爬虫
  - Java爬虫
  - Jsoup
toc: true
date: 2020-01-03 22:11:13
---
## Jsoup技术入门

在使用Jsoup之前,解析响应通常使用字符串和正则表达式来获取目标数据, 但是正则表达式还是相对麻烦.

Jsoup是一款Java的HTML解析器,可以直接解析某个URL地址,HTML内容. Jsoup提供了丰富的API,可以向前端一样通过DOM等操作方法获取目标数据

### 简单实例

虽然Jsoup也可以直接通过URL抓取内容, 但不用来做爬虫,因为开发中往往需要使用多线程,连接池等技术, Jsoup的支持并不好, 所以Jsoup通常依赖解析数据

#### 解析URL

```java
@Test
public void testUrl() throws IOException {
    // Jsoup解析URL地址
    Document document = Jsoup.parse(new URL("https://www.baidu.com"),3000);
    // 使用标签选择器获取标题
    String title = document.getElementsByTag("title").first().text();
    System.out.println(title);
}
```

#### 解析String

```java
@Test
public void testString(){
    // 封装好的HttpClient爬虫
    String html=SpiderFirst.getSpider("https://www.baidu.com");
    Document document = Jsoup.parse(html);
    String title = document.getElementsByTag("title").first().text();
    System.out.println(title);
}
```

#### 解析File

```java
@Test
public void testfile() throws IOException {
    Document document = Jsoup.parse(new File("C:\\Users\\yanchengjie\\Desktop\\baidu.html"),"utf8");
    String title = document.getElementsByTag("title").first().text();
    System.out.println(title);
}
```

#### DOM操作

```java
@Test
public void testDOM() {
    // 封装好的HttpClient爬虫
    String html = SpiderFirst.getSpider("http://www.itcast.cn/");
    Document document = Jsoup.parse(html);
    Element element = document.getElementById("webim");
    System.out.println(element.toString());
    // 获取id值
    String id = element.id();
    String id_ = element.attr("id");
    System.out.println(id + " " + id_);
    // 获取className
    String className = element.child(0).className();
    String className_ = element.child(0).attr("class");
    System.out.println(className+" "+className_);
    // 获取所有元素
    Attributes attrs = element.child(0).child(0).child(0).attributes();
    System.out.println(attrs.toString());
    // 获取文本内容
    String text = element.child(0).child(0).child(0).text();
    System.out.println(text);
}
```

#### 选择器操作

```java
@Test
public void testSelector(){
    // 封装好的HttpClient爬虫
    String html = SpiderFirst.getSpider("http://www.itcast.cn/");
    Document document = Jsoup.parse(html);
    // 使用标签选择器
    Elements elements = document.select("span");
    for(Element element: elements){
        System.out.println(element.text());
    }
    // 使用id选择器
    Element element = document.select("#webim").first();
    System.out.println(element.toString());
    // 使用类选择器
    Elements elements1 = document.select(".a_default");
    for(Element element1: elements1){
        System.out.println(element1.text());
    }
    // 元素选择器
    Elements elements2 = document.select("[class=slogan]");
    for(Element element2: elements2){
        System.out.println(element2.toString());
    }
    // 组合选择器
    Elements elements3 = document.select("img.slogan");
    for(Element element3: elements3){
        System.out.println(element3.toString());
    }

    Elements elements4 = document.select(".box_hd > h2");
    for(Element element4: elements4){
        System.out.println(element4.toString());
    }
}
```
