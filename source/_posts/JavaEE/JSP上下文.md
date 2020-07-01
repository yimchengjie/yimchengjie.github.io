---
title: Servlet上下文
date: 2017-12-25 18:47
categories:
  - JavaEE
tags:
  - JavaEE
  - JSP
toc: true
---

#### 什么是上下文

上下文 ServletContext 对象是用来存储全局范围信息的对象;换句话说,一个 Web 应用只有唯一一个上下文对象.

- 当服务器启动的时候，就会为每一个应用创建一个上下文对象；
- 当服务器关闭的时候，上下文对象就销毁；

#### Servlet 中的 ServletContext 接口

|                            方法声明                            |                                       方法描述                                       |
| :------------------------------------------------------------: | :----------------------------------------------------------------------------------: |
| java.io.InputStream getResourceAsStream(java.lang.String path) | 将 path 所代表的资源以输入流返回，可以进一步进行读操作；可以用来读取服务器端的文件； |
| RequestDispatcher getRequestDispatcher(java.lang.String path)  |               返回 RequestDispatcher 对象，路径是相对于上下文路径的；                |

#### 上下文获取方法

Servlet 规范中的多个接口中都定义了`getServletContext`方法获得上下文对象

#### 上下文参数

- 在 web.xml 中可以配置上下文参数，使用`ServletContext`中的`getInitParameter`方法可以获取该参数；【之前学习过的 Servlet 初始化参数，只能在当前 Servlet 中使用】
- 上下文参数存储在上下文对象，所以应用下所有组件都可以使用；
- 获取上下文参数：

  ```html
  <context-param>
    <param-name>version</param-name>
    <param-value>2.0</param-value>
  </context-param>
  ```

  ```java
  //返回ServletContext对象
  ServletContext ctxt=this.getServletContext();
  //获取上下文参数
  String version=ctxt.getinitParameter("version");
  System.out.println("上下文参数version的值:"+version);
  ```

#### 利用 ServletContext 在应用中共享数据

|                           方法声明                           |                   方法描述                   |
| :----------------------------------------------------------: | :------------------------------------------: |
| void setAttribute(java.lang.String name, java.lang.Object o) | 将任意类型对象设置为上下文属性，指定一个名字 |
|     java.lang.Object getAttribute(java.lang.String name)     |        通过属性的名字，获取属性的值；        |
|         void removeAttribute(java.lang.String name)          |          通过属性的名字，删除属性；          |

#### 四大作用范围

在 Web 应用中，有四大作用域范围

- 页面范围`PageContext`：一个 Servlet 或 JSP 文件；
- 请求范围`ServletRequest`：一次请求中可以访问多个 Servlet 或 JSP； 访问的 Servlet 或 JSP 能够包含其他资源；
- 会话范围`HttpSession`：一次会话中可以包含多个请求；
- 上下文范围`ServletContext`：上下文包含所有会话；
