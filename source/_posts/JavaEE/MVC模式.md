---
title: MVC模式
date: 2018-01-19 14:15
categories:
  - JavaEE
tags:
  - JavaEE
toc: true
---

#### MVC 模式基本概念

MVC（Model-View-Controller）是一种软件架构设计模式，最初应用在桌面应用程序；

MVC 模式将软件的代码按照模型（M）、视图（V）、控制器（C）三部分组织

MVC 模式构建应用的优势:

- 耦合性低：视图层和业务层分离，耦合性降低，可以独立修改；
- 重用性高：可以用不同的视图访问模型部分，实现在不同终端上访问应用；
- 可维护性高：视图与业务分离，降低了维护成本；

#### MVC 模式中的三个角色

在控制器和视图之间共享数据:

1. 在控制器和视图之间，常常需要共享数据；例如从数据查出来的商品列表信息，需要从控制器发送到视图；
2. Servlet 和 JSP 之间共享数据一般使用请求、会话、上下文范围的属性进行；
3. HttpServletRequest/HttpSession/ServletContext 接口中都定义了存取、查询、删除属性的方法【前面已经学习过】；
4. 使用原则：尽量用范围小的属性，即，请求范围内共享即可就用请求，以此类推；否则会造成资源浪费，降低安全性；

#### redirect\forward\include 几种跳转方式的功能与差异

MVC 模式中，控制器和视图之间需要进行跳转，Servlet 规范中，有三种跳转方式：

1. redirect：调用响应接口的 sendRedirect 方法，响应重定向，相当于重新请求新的资源，当前请求对象不会到目标资源；
2. forward: 调用请求转发器接口的 forward 方法，请求转发，将当前的请求、响应对象转发到目标资源；(最常用)
3. include：调用请求转发器接口的 include 方法，动态包含，将目标资源的请求、响应对象包含到当前资源；

#### forword 带来的重复提交问题

- 使用 forward 转发请求后，再次刷新当前页面，会进行重复提交；
  - 例如：使用 LoginServlet 进行登录，成功后跳转到 loginsuccess.jsp 页面：
- 刷新当前页面，再次进行了登录

- 为了能够解决重复提交问题，关键在于：能够标志一次提交，从而识别出该提交已经处理；
  1. 步骤一：在 JSP 中记录一个随机数，称为令牌（token），存储在 session 中
  `<%session.setAttribute("token",System.nanoTime())+""%>`
  2. 步骤二：将 token 值作为表单的一个隐藏域
     `<input type="hidden" name="token" value="<%=session.getAttribute("token")%>" >`
  3. 步骤三：在 LoginServlet 中获取 token 值，并进行判断

      ```java
      //取出存储在请求参数中的token
      String requestToken = request.getParameter("token");
      //取出存储在session中的token
      String sessionToken = (String)request.getSession().getAttribute("token");
      ....
      ```
  
  4. 步骤四：将 token 值从会话中删除

     ```java
     request.getSession().removeAttribute("token");
     ```
