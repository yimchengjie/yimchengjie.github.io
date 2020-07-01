---
title: JSP
date: 2017-12-15 14:37
categories:
  - JavaEE
tags:
  - JavaEE
  - JSP
toc: true
---

#### 什么是 JSP

JSP（Java Server Page）是 Java 服务端的页面，所以它是动态的，它是需要经过 JDK 编译后把内容发给客户端去显示，我们都知道，Java 文件编译后会产生一个 class 文件，最终执行的就是这个 class 文件。翻译和编译的过程遵守 Servlet 规范，因此说==JSP 的本质也是 Servlet==；
html 和 jsp 的表头不一样，这个是 JSP 的头`<%@ page language=”java” import=”java.util.*” pageEncoding=”gbk”%>`在表头中有编码格式和倒入包等。也是很好区分的，在 jsp 中用`<%%>`就可以写 Java 代码了，而 html 没有`<%%>`。

<u>**简单说，jsp 在后台通过服务器解析为相应的 html，然后在供浏览器识别显示。**</u>

#### 为什么要用 JSP

Servlet 生成动态页面比较繁琐，使用 JSP 生成动态页面比较便捷，因为其中的静态内容可以使用 HTML 生成；

#### JSP 元素

- 脚本元素可以用来包含任意 Java 代码,格式为：`<%Java代码%>`

  - 服务器翻译脚本元素时，将把其中 Java 代码直接翻译到`jspService`方法中，如果语法错误，将在浏览器中提示错误；

- 表达式元素用来向页面输出动态内容;格式为：`<%=Java代码%>`

  - 服务器翻译表达式元素时，将把其中 Java 代码部分的返回值使用 out.write 语句输出

- 模块元素指 JSP 中静态 HTML 或者 XML 内容

- 注释元素有三种情况：

  - 格式为`<%--JSP注释--%>`；JSP 的注释只有在源代码中可见，翻译时已经忽略； + 在 JSP 中，除了使用 JSP 注释外，还可以使用 HTML 注释，`<!--HTML注释-->`，HTML 注释会被返回到客户端，但是不显示到页面中； + JSP 中的 Java 代码部分，可以使用 Java 注释；Java 注释会翻译到.java 文件中，但是编译时忽略；

- 声明元素:
  - 如果需要在 JSP 文件中定义类的成员变量或方法，可以使用声明元素，格式为`<%! 声明语句%>`

    ```jsp
    <%! private String path="WEB-INF"; public void readPropertiesFile(){}>
    ```

  - 声明元素被翻译到 Java 类中，而不是\_jspService 方法中；

#### 内置对象

内置对象指的是在 JSP 中**可以直接使用的对象，不需要声明**，直接使用固定的名字使用即可；例如`<%=request.getRemoteAddr()%>`中的`request`就是内置对象；

jsp 中共有 9 种对象

1. `request`：用户端请求，此请求会包含来自 GET/Post 请求的参数；
2. `response`：网页传回用户端的回应。
3. `pageContext`：页面的属性是在这里管理
4. `session`：与请求有关的回话期
5. `application` ：Servlet 正在执行的内容
6. `out` ：用来传递回应的输出
7. `config` ：servlet 的构架部件
8. `pagejsp`网页本身
9. `exception` ：针对错误的网页。未捕捉的例外

#### Servlet 和 JSP 的作用

实际应用中，Servlet 是不会用来生成动态页面的，而是会用来接收来自 JSP 的请求，处理请求，然后调跳转到 JSP 页面把结果显示给客户端看；

#### Servlet 与 JSP 之间的跳转方式

1. 跳转方式一:**响应重定向**,响应接口中提供了该方法

   - `void sendRedirect(java.lang.String location)`:响应重定向到 location，相当于客户端重新请求 location 所在的资源；

   - 第一个 JSP 页面发送请求`request`到`Servlet`,`Servlet`接收请求后,响应`response`重定向到目标 JSP 页面,但是请求并没有传递过来.(**重定向相当于是产生一个新的请求**)

2. 跳转方式二:**请求转发**,RequestDispatcher 接口定义了请求转发的方法
   - `forward(ServletRequest request, ServletResponse response)`:将请求转发到服务器上的其他资源，包括其他的 Servlet，JSP 等；
   - 要使用 forward 方法，需要先获得 RequestDispatcher 对象；请求接口(request)中提供了获得该对象的方法：
     - `RequestDispatcher getRequestDispatcher(java.lang.String path)`:使用 path 返回一个 RequestDispatcher 对象
   - 请求转发把请求对象发送到了目标 JSP 页面,因此**目标页面可以获得上一个页面的请求对象.**

#### 请求属性的使用

如果需要在 Servlet，JSP 之间跳转时，同时把一些自定义的、或者通过数据库查询的、或者调用其他资源获得的数据传递到下一个资源时，就可以把这些数据设置为请求的属性即可。

请求接口中定义了一系列与属性有关的方法。
|方法声明|方法描述|
|:------:|:-----:|
|void setAttribute(java.lang.String name, java.lang.Object o)|将任意类型对象设置为请求的属性，指定一个名字；|
|java.lang.Object getAttribute(java.lang.String name)|通过属性的名字，获取属性的值；|
|void removeAttribute(java.lang.String name)|通过属性的名字，删除属性；|
可以将数据封装进请求对象中,在前后端传递
例如:后台 Servlet 中将数据保存进 request(请求)中,跳转到前台 JSP 后,JSP 可以用`<%=request.getAttribute("name")%>`来获取(直接输出)也可以**保存为变量(前提是需要强转**,因为获取的是 Object 类型)
