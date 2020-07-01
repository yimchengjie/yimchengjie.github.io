---
title: 初探Java爬虫
categories:
  - 爬虫
tags:
  - 爬虫
  - Java爬虫
toc: true
date: 2019-12-27 21:09:53
---
## 初探Java爬虫

既然学的是Java,那就用Java来学爬虫.

### Demo

```java
public class spiderDemo {
    public static void main(String[] args) {
        // 爬取的目标url
        String url = "https://www.baidu.com";
        // 空字符串,用于后续接收内容
        StringBuffer result = new StringBuffer();
        BufferedReader in = null;
        try{
            // 将字符串url转换成URL类型
            URL readUrl = new URL(url);
            // 进行链接初始化
            URLConnection connection = readUrl.openConnection();
            // 开始连接
            connection.connect();
            // 创建一个输入流来读取响应内容
            in = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            // 创建读取临时变量(读取缓存)
            String tempStr = "";
            while ((tempStr = in.readLine())!=null){
                result.append(tempStr);
            }
            System.out.println(result);
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            System.out.println("爬取结束");
        }
    }
}
```

程序运行结果
![爬取结束](/爬取结束.png)

可以看到, 上述程序连接到URL,然后读取了响应, 也就是baidu首页的源码

那么如何获取更细的内容呢.

### 正则表达式的应用

```java
 public static void main(String[] args) {
        SpiderDemo spiderDemo = new SpiderDemo();
        // 爬取的目标url
        String url = "https://www.baidu.com";
        // 将爬取功能封装
        String result = spiderDemo.spiderMan(url);
        // 定义样式模板,匹配<img>标签
        String patternStr = "<img.*src\\s*=\\s*(.*?)[^>]*?>";
        // 要匹配的目标字符串
        String targetStr = result;
        // 将样式模板字符串转型
        Pattern pattern = Pattern.compile(patternStr);
        // 定义匹配对象进行匹配
        Matcher matcher = pattern.matcher(targetStr);
        // 如果匹配成功
        while (matcher.find()){
            System.out.println("匹配成功");
            System.out.println(matcher.group());
        }
    }
```

正则表达式是爬虫获取精确内容的重要基础
