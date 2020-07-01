---
title: Selenium在爬虫中的应用
categories:
  - 爬虫
tags:
  - 爬虫
  - Java爬虫
  - Selenium
toc: true
date: 2020-01-30 21:30:22
---

## Selenium 在爬虫中的应用

---

### 什么是 Selenium

Selenium 是目前用的最广泛的 Web UI 自动化测试框架。
核心功能是在多个浏览器上进行自动化测试
支持跨平台,支持多种语言.

### 在 Java 中使用 Selenium

在学习爬虫实践的过程中,发现很多网站使用 js 渲染,且 api 调用需要秘钥,导致无法爬取, 这时候就用到了 Selenium, 通过 Selenium 我们能得到经过浏览器渲染后产生的 HTML 文档.毕竟浏览器最终呈现的都是解析后的. 这样我们就能获得完整的 HTML 文档了

1. 在 maven 中导入包

    ```xml
    <!--selenium-->
    <dependency>
      <groupId>org.seleniumhq.selenium</groupId>
      <artifactId>selenium-java</artifactId>
      <version>3.4.0</version>
    </dependency>
    ```

2. 使用浏览器调试工具

    要使用 WebUI 自动化测试, 需要使用浏览器的驱动
    工具系统安装的浏览器,选择对应版本的驱动
    [谷歌浏览器驱动下载地址](http://chromedriver.storage.googleapis.com/index.html)

3. 使用 WebDriver

    ```java
    // 1. 得到WebDriver实例
    public static WebDriver initWebDriber(String diverPath) {
        System.setProperty("webdriver.chrome.driver", diverPath);// diverPath是chromedriver服务地址
        WebDriver webDriver = new ChromeDriver();
        return webDriver;
    }

    // 2. 打开页面
    webDriver.get(url);

    // 3. 进行操作(比如登录,打开隐藏窗口等)
    driver.findElement(By.className("UG_box"));//找到要操作的element
    webElement.click(); //使用获得的element进行点击操作

    // 4. 下载页面
    String html = webDriver.getPageSource();
    // 得到String格式的HTML文档
    ```

#### findElement/findElements详解

通过findElement/findElements可以定位element,获取element进行后续操作
通常使用By与之结合, By是一个类,里面封装了获取element的方法.

1. By.id()

    根据标签的**id**获取

2. By.name()

    通过标签的**name**获取

3. By.tagName()

    根据 标签 获取
  
4. By.className()

    根据标签中**类class**的**值**获取

5. By.lintText()

    通过页面中**超链接包含的文字**来定位

6. By.xpath()

    通过Xpath语法定位

#### 对于浏览器窗口的操作

有时候不是所有操作都能用HTML定位来做,比如浏览器弹出的各种窗口

1. 操作弹出窗口

    ```java
    // 获取弹出窗口
    Alert al = driver.switchTo().alert();
    // 选择确定按钮
    al.accept();
    // 选择取消按钮
    al.dismiss();
    ```

2. 浏览器全屏

    ```java
    driver.manage().window().maximize();
    ```
  
3. 关闭浏览器窗口

    ```java
    driver.quit();

    driver.close();
    ```
  
4. 刷新/前进/回退

    ```java
    // 刷新
    driver.navigate().refresh();
    // 前进
    driver.navigate().forward();
    // 回退
    driver.navigate().back();
    ```
  
#### 程序等待方式

1. sleep();

    强制等待,设置睡眠时间
  
2. implicitlyWait()：隐式等待，等待元素被发现、命令完成，超出了设置的时间则跑出异常

    ```java
    WebDriver driver = new ChromeDriver();
    //设置脚本在查找元素时的最大等待时间
    driver.manage().timeouts().implicitlyWait(15, TimeUnit.SECONDS);
    ```

3. WebDriverWait

    ```java
    //设置等待的时长，最长10s
    WebDriverWait wait = new WebDriverWait(driver, 10);  
    wait.until(ExpectedConditions.presenceOfElementLocated(By.id("app"))));
    ```
