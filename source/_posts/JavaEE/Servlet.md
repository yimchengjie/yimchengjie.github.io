---
title: Servlet
date: 2017-12-06 17:12
categories:
  - JavaEE
tags:
  - JavaEE
  - Servlet
toc: true
---

#### 什么是 Servlet

Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

#### Servlet 的线程特性

Web 应用服务器(Tomcat)将为每个客户端的连接启动一个线程来服务
第一次访问 Servlet 时，服务器将创建一个该 Servlet 类的对象，并调用 doXXX 方法生成响应；多个客户端访问同一个 Servlet 时，不再创建新的对象，而是共用同一个 Servlet 对象。可以说，Servlet 是多线程单实例的。

#### Servlet 请求和响应接口

通过浏览器提交给服务端的所有数据,都称为**请求数据**

通过服务器返回给客户端的所有数据,都称为**响应数据**

ServletAPI 中,定义了请求和响应接口,用来封装和操作请求和响应数据

- 请求接口:
  - `javax.servlet.ServletRequest`
  - `javax.servlet.HttpServletRequest`
- 响应接口:
  - `javax.servlet.ServletResponse`
  - `javax.servlet.HttpServletResponse`

Servlet 类使用 doXXX 方法提供服务,这些方法继承于`HttpServlet`
doXXX 方法中都有两个参数,分别是请求和响应;
|方法|参数|作用|
|:--:|:--:|:--:|
|void doDelete|(HttpServletRequest request,HttpServletResponse response)|用来处理一个 HTTP DELETE 操作,这个操作允许客户端请求从服务器上删除 URL|
|void doGet|(HttpServletRequest request, HttpServletResponse response)|用来处理一个 HTTP GET 操作。这个操作允许客户端简单地从一个 HTTP 服务器“获得”资源|
|void doHead|(HttpServletRequest request, HttpServletResponse response)|用来处理一个 HTTP HEAD 操作。默认的情况是，这个操作会按照一个无条件的 GET 方法来执行|
|void doOptions|(HttpServletRequest request, HttpServletResponse response)|用来处理一个 HTTP OPTION 操作。这个操作自动地决定支持哪一种 HTTP 方法。例如，一个 Servlet 写了一个 HttpServlet 的子类并重载了 doGet 方法，doOption 会返回下面的头： Allow:GET,HEAD,TRACE,OPTIONS|
|void doPost|(HttpServletRequest request, HttpServletResponse response)|用来处理一个 HTTP POST 操作。这个操作包含请求体的数据，Servlet 应该按照他行事。|
|void doPut|(HttpServletRequest request, HttpServletResponse response)|用来处理一个 HTTP PUT 操作。这个操作类似于通过 FTP 发送文件。|
|void doTrace|(HttpServletRequest request, HttpServletResponse response)|用来处理一个 HTTP TRACE 操作。这个操作的默认执行结果是产生一个响应，这个响应包含一个反映 trace 请求中发送的所有头域的信息。|
|long getLastModified|(HttpServletRequest request)|返回这个请求实体的最后修改时间。|
|void service|(HttpServletRequest request, HttpServletResponse response)|这是一个 Servlet 的 HTTP-specific 方案，它分配请求到这个类的支持这个请求的其他方法。当你开发 Servlet 时，在多数情况下你不必重载这个方法。|
**_也就是说：服务器会创建请求对象和响应对象传递给 doXXX 方法，doXXX 方法中可以直接使用请求和响应对象;_**
**doXXX 方法中可以使用方法参数 request，response 去调用请求和响应接口中的方法；**

#### 利用 Servlet 对客户端不同方式请求作出动态响应

客户端访问服务器端 Servlet 的三种方式:

1. 直接从地址栏输入 URL 访问；是 GET 方式，调用 doGet 方法;
2. 在网页中点击超级链接访问；是 GET 方式，调用 doGet 方法;
3. 在网页中通过表单提交访问；取决 form 的 method 属性的值，默认是 get，指定为 post 时，调用 doPost 方法;

Servlet 中获取请求参数的方法:

1. 可以在 URL 后使用 name=value&name=value 的形式传递，例如：

   ```html
   <a href=“TestPramServlet?page=1&author=wangxh”>
   <!-- 传递两个请求参数，名字分别为page和author，值分别为1和wangxh；-->
   ```

2. 可以在使用表单提交，表单中的元素值将作为请求参数传递，元素的 name 是参数名字，value 的值是参数的值

当客户端请求服务器端的 Servlet 时，请求参数都会被发送到服务器，服务器会将请求参数封装到**请求对象**中；

#### Servlet 初始化参数

1. 如果某个 Servlet 需要使用一些可以配置的参数，可以在 web.xml 进行配置，称为初始化参数；
2. 这些参数在服务器初始化 Servlet 实例时被初始化到配置信息中，可以在 Servlet 中获取并使用；
3. 一个 Servlet 可以配置多个初始化参数，所有的初始化参数只能在当前 Servlet 类中使用；

#### Servlet 加载启动选项

1. 默认情况下，只有当第一次访问 Servlet 时，服务器才会初始化 Servlet 实例；
2. 如果需要更早实例化 Servlet，可以在 web.xml 中进行配置，使得在启动容器的时候就能初始化 Servlet 实例；

#### Servlet 配置中通配符\*的用法

- _.扩展名 ： 比如 _.do、\*.action
- 以 / 开头，同时以 /_ 结尾，比如 /_ 、/admin/\*

#### web.xml 中首页及错误页面等其他配置信息

- **配置默认页面**:当不指定具体访问路径时,默认访问默认页面

```html
<welcome-file-list>
  <welcome-file>index.html</welcome-file>
  <welcome-file>index.htm</welcome-file>
  <welcome-file>index.jsp</welcome-file>
  <welcome-file>default.html</welcome-file>
  <welcome-file>default.htm</welcome-file>
  <welcome-file>default.jsp</welcome-file>
</welcome-file-list>
```

- **配置错误页面**:当应用中出现响应错误或者异常时,可以跳转到错误页面;

#### Servlet 中获取请求头属性的方法

客户端请求服务端的 Servlet,会传递给服务器一系列的 HTTP 请求头属性,请求接口中定义了系列方法获取请求属性
