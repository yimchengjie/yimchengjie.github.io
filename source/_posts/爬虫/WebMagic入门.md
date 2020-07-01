---
title: WebMagic入门
categories:
  - 爬虫
tags:
  - 爬虫
  - Java爬虫
  - WebMagic
toc: true
date: 2020-01-17 21:39:19
---

## WebMagic 入门

WebMagic 采用完全模块化的设计，有着强大的可扩展性,基于 HttpClient 和 Jsoup
核心简单但却涵盖了爬虫的全部流程。
有着丰富的页面抽取 API
支持多线程分布式，支持爬取动态 js
没有框架依赖

### WebMagic 架构

WebMagic 框架包含四个组件，PageProcessor、Scheduler、Downloader 和 Pipeline。这四大组件对应爬虫生命周期中的处理、管理、下载和持久化等功能。
这四个组件都是 Spider 中的属性，爬虫框架通过 Spider 启动和管理。

1. Downloader: 负责从互联网上下载页面,以便后续处理.一般无需自己实现
2. Scheduler: 负责管理待抓取的 URL,以及一些去重工作.一般无需自己实现
3. PageProcessor: 负责解析页面,抽取目标信息,以及发现新 URL,需要自定义
4. Pipeline: 负责抽取结果的处理,包括计算,持久化等.

#### 数据流转的对象

1. Request: 是对 URL 地址的一层封装,一个 Request 对象对应一个 URL 地址
2. Page: 代表了从 Downloader 下载到的一个页面,可能是 HTML 页面也可能是其他文本内容(XML,JSON)
3. ResultItems: 相当于一个 MAP,保存了 PageProcessor 处理的结果,供 Pipeline 使用

### 简单实例

#### 环境配置

maven 引入依赖

```xml
<!--webmagic-->
<dependency>
  <groupId>us.codecraft</groupId>
  <artifactId>webmagic-core</artifactId>
  <version>0.7.3</version>
</dependency>
<dependency>
  <groupId>us.codecraft</groupId>
  <artifactId>webmagic-extension</artifactId>
  <version>0.7.3</version>
  <exclusions>
    <exclusion>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

#### Demo

在 WebMagic 中,实现爬虫只需要编写一个类,实现 PageProcessor 接口即可,这个类基本上包含了抓取一个网站,所需要写的所有代码

```java
public class TestPageProcessor implements PageProcessor {

  // 部分一：抓取网站的相关配置，包括编码、抓取间隔、重试次数等
  private Site site = Site.me().setRetryTimes(3).setSleepTime(1000);

  @Override
  // process是定制爬虫逻辑的核心接口，在这里编写抽取逻辑
  public void process(Page page) {
      // 部分二：定义如何抽取页面信息，并保存下来
      page.putField("author", page.getUrl().regex("https://github\\.com/(\\w+)/.*").toString());
      page.putField("name", page.getHtml().xpath("//h1[@class='entry-title public']/strong/a/text()").toString());
      if (page.getResultItems().get("name") == null) {
          //skip this page
          page.setSkip(true);
      }
      page.putField("readme", page.getHtml().xpath("//div[@id='readme']/tidyText()"));

      // 部分三：从页面发现后续的url地址来抓取
      page.addTargetRequests(page.getHtml().links().regex("(https://github\\.com/[\\w\\-]+/[\\w\\-]+)").all());
  }

  @Override
  public Site getSite() {
      return site;
  }

  public static void main(String[] args) {
      Spider.create(new TestPageProcessor())
              //从"https://github.com/code4craft"开始抓
              .addUrl("https://github.com/code4craft")
              //开启5个线程抓取
              .thread(5)
              //启动爬虫
              .run();
  }
}
```

#### 注解模式开发

```java
@TargetUrl("https://github.com/\\w+/\\w+")   //要抓取的目标url
// 在TargetUrl页面得到的URL,只要符合TargetUrl的格式,也会被下载
//  TargetUrl还支持定义sourceRegion，这个参数是一个XPath表达式，指定了这个URL从哪里得到——不在sourceRegion的URL不会被抽取。
@HelpUrl("https://github.com/\\w+")   // 为了访问目标url需要访问的url
/*
    对于博客页，HelpUrl是列表页，TargetUrl是文章页。
    对于论坛，HelpUrl是帖子列表，TargetUrl是帖子详情。
    对于电商网站，HelpUrl是分类列表，TargetUrl是商品详情。
 */
public class GithubRepo {

    //@ExtractBy注解主要作用于字段，它表示“使用这个抽取规则，将抽取到的结果保存到这个字段中”。
    @ExtractBy(value = "//h1[@class='entry-title public']/strong/a/text()", notNull = true)
    private String name;

    @ExtractByUrl("https://github\\.com/(\\w+)/.*")
    private String author;

    @ExtractBy("//div[@id='readme']/tidyText()")
    private String readme;

    public static void main(String[] args) {
        OOSpider.create(Site.me().setSleepTime(1000)
                , new ConsolePageModelPipeline(), GithubRepo.class)
                .addUrl("https://github.com/code4craft").thread(5).run();
    }
}
```

##### 一个完整流程

1. 编写爬虫

   ```java
   @TargetUrl("https://github.com/\\w+/\\w+")
   @HelpUrl("https://github.com/\\w+")
   public class GithubRepo {

       @ExtractBy(value = "//h1[@class='entry-title public']/strong/a/text()", notNull = true)
       private String name;

       @ExtractByUrl("https://github\\.com/(\\w+)/.*")
       private String author;

       @ExtractBy("//div[@id='readme']/tidyText()")
       private String readme;
   }
   ```

2. 启动

   ```java
   public static void main(String[] args) {
       // OOSpider是入口, 参数分别为, 请求参数, 结果处理链, 爬虫类
       OOSpider.create(Site.me().setSleepTime(1000)
               , new ConsolePageModelPipeline(), GithubRepo.class)
               .addUrl("https://github.com/code4craft").thread(5).run();
   }
   ```
