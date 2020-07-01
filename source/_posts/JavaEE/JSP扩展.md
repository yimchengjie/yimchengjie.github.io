---
title: JSP扩展
date: 2017-12-29 10:25
categories:
  - JavaEE
tags:
  - JavaEE
  - JSP
toc: true
---

#### JSP 内置对象

- 内置对象指的是服务器已经创建好的对象，可以直接使用；
- 9 个内置对象:

  - request
  - response
  - out
    - out 的类型是 JspWriter
    - out 可以用来输出内容到客户端，但是程序员一般不会使用，因为直接使用<%=%>即可以实现输出；
  - page + page 即当前类对象 + page 也很少使用，与 this 相同
  - pageContext + 其他多数内置对象都是通过它获得

    ```java
    application = pageContext.getServletContext();
    config = pageContext.getServletConfig();
    session = pageContext.getSession();
    out = pageContext.getOut();
    ```

  - pageContext 对象是 JSP 中一个非常重要的对象，是`javax.servlet.jsp.PageContext`类型的对象，指的是页面的上下文，封装了其他的内置对象，同时代表的是四大作用域【页面、请求、会话、上下文】中的页面作用域，也可以在页面上下文范围添加属性，`PageContext`中与属性相关方法如下：

      |                           方法声明                           |                 方法描述                 |
      | :----------------------------------------------------------: | :--------------------------------------: |
      | void setAttribute(java.lang.String name, java.lang.Object o) | 将任意类型对象设置为属性，指定一个名字； |
      |     java.lang.Object getAttribute(java.lang.String name)     |      通过属性的名字，获取属性的值；      |
      |         void removeAttribute(java.lang.String name)          |        通过属性的名字，删除属性；        |

  - session + session 是 JSP 中的另一个内置对象，是`HttpSession`类型的对象，可以在 JSP 中调用 HttpSession 接口中的任何方法；默认存在 + application + application 是 JSP 中的另一个内置对象，是`ServletContext`类型的对 象，可以在 JSP 中调用`ServlletContext`接口中的任何方法； + exception + 内置对象 exception 比较特殊，默认情况下不存在；只有当 JSP 中使用指令指定该页面作为错误页面使用时才会翻译生成该内置对象。 + config + 在 JSP 中可以直接使用 config 对象调用 ServletConfig 接口中任意方法，例如，可以在 web.xml 中对 JSP 配置初始化参数，与前面学习的 Servlet 初始化参数相同的含义：

#### 指令与动作

- JSP 可以通过指令元素而影响容器翻译生成 Java 类的整体结构；
- 指令的语法为：`<%@ directive {attr=“value”}* %>`；
- 其中，directive 为指令名，attr 指该指令对应的属性名，一个指令可能有多个属性；
  JSP 中常用的指令有三个：page、include、taglib，前两个常用
- **page 指令**作用于整个 JSP 页面，可以将指令放在 JSP 页面任何一个位置;

  - import 属性:用来引入 JSP 文件需要使用的类； + 可以使用逗号同时引入多个包，也可以在一个 JSP 文件中多次使用 import； + 值得注意的是，import 是 page 指令中唯一一个可以在一个 JSP 文件中多次出现的属性，其他属性在一个 JSP 文件中只能出现一次；
  - pageEncoding 属性:用来设置 JSP 文件的页面编码格式； + page 指令的 session 属性：用来设置 JSP 页面是否生成 session 对象。该属性默认值为 true，可以设置成 false。 + session 属性值设置为 false 后，该 JSP 翻译生成的类中将没有内置对象 session，该 JSP 不参与会话。
  - errorPage 属性:设置 JSP 页面的错误页面。当 JSP 中发生异常或错误却没有被处理时，容器将请求转发到错误页面； + 访问该页面将发生数学异常，而且并没有对异常进行处理，那么将跳转到错误页面 error.jsp
  - isErrorPage 属性默认值是 false，可以设置为 true。在 JSP 的错误页面中，将 isErrorPage 设置为 true，则该页面翻译生成的 Java 类中，将生成 exception 内置对象。在 error.jsp 中加入代码：`<%@page isErrorPage="true"%>` + 上述代码将 error.jsp 页面设置为错误页面，所以，在 error.jsp 翻译生成的 Java 类中的\_jspService 方法中将生成 exception 内置对象
    - 注意：即使一个页面没有设置 isErrorPage=“true”，也可以作为错误页面使用，区别在是否有内置对象 exception 内置对象产生。

- **include 指令**是 JSP 中另外一个常用指令，用来静态包含其他页面；

  - 在翻译期间，把包含的页面也翻译到当前页面的 Java 文件中，也就是 Java 源文件即实现“二合一”；
  - `<%@include file="copyright.jsp"%>`

- **include 动作标签**:

  - JSP 规范中定义了一系列的标准动作。Web 容器按照规范进行了实现，可以解析并执行标准动作；
  - 标准动作使用标准的 XML 语法。
  
    ```jsp
    <jsp:action_name attribute1="value1" attribute2="value2">
    </jsp:action_name>
    ```
  
  - 其中 action_name 表示标准动作的名字，attribute1 和 attribute2 是标准动作的若干个属性；
  - include 标准动作:`<jsp:include>`是动态包含，即在运行期访问被包含的页面，并将响应结果同包含页面的响应结果合并，生成最终响应。类似在 Servlet 中调用`RequestDispatcher`的`include`方法进行包含。

- **include 标准动作和 include 指令的差异**;

  - include 标准动作与 include 指令都是实现包含其他页面的功能;
  - include 标准动作的属性是 page，实现动态包含，发生在请求阶段；
  - include 指令的属性是 file，实现静态包含，发生在翻译阶段。

- include 其他动作
  - forward 动作：在 JSP 页面中进行请求转发，如下代码所示：
    `<jsp:forward page=“loginsuccess.jsp"> </jsp:forward>`
  - param 动作：往往作为子动作使用，为 forward 和 include 动作传递参数，如下代码所示：
    `<jsp:forward page="copyright.jsp"> <jsp:param name="author" value="etc"/> </jsp:forward> <jsp:include page="copyright.jsp"> <jsp:param name="author" value="etc"/> </jsp:include>`
  - 上述代码使用 param 为 forward 和 include 动作传递参数，参数将被作为请求参数传递。
  - 使用标准动作时，一定注意正确结束标准动作，如`<jsp:include>`是标准动作的开始，一定要对应结束标记，如`</jsp:include>`。

#### JavaBean

JavaBean 是用 Java 语言描述的软件组件模型，实际上是一个 Java SE 的类，这些类遵循一定的编码规范：

1. 必须是 public 类 ；
2. 必须有一个无参的 public 的构造方法；
3. 返回属性的方法为 getXxxx()格式 ；
4. 设置属性的方法为 setXxx(形式参数)格式；

JSP 中还提供了 3 个与 JavaBean 有关的动作；

1. useBean动作：`<jsp:useBean  id=“” class=“” scope=“”>`
    - useBean 标准动作用来使用 JavaBean 对象，JavaBean 对象是某一范围（用 scope 指定）的属性；
    - Java Bean 对象名字用 id 指定，类型用 class 指定。如果对应范围没有该属性，则调用 class 指定类的无参构造方法，创建一个该类的对象，并将该对象存储为 scope 内的一个属性，属性名为 id；
    - 其中 scope 有四种：page、request、session、application，分别为 PageContext 范围、HttpServletRequest 范围、HttpSession 范围、ServletContext 范围。如果不指定 scope 的值，默认为 page 范围。
2. setProperty 动作：`<jsp:setProperty name=“” property=“” param|value=“”/>`
    - setProperty 标准动作可以用来对 JavaBean 对象的属性赋值，替代调用 setXxxx 方法；
    - setProperty 的 name 属性表示 JavaBean 对象的 id 值，property 表示需要调用的 setXxx 方法中的 Xxx 部分，将首字母变小写。比如需要调用 setCustname 方法，则 property 即为 Custname 首字母变小写，即 custname；
    - 如果 setXxx 方法的参数是某一个请求参数的值，则使用 param 属性指定请求参数名字即可；
    - 如果 setXxx 方法的参数是一个常量，则使用 value 属性指定即可。
    - 同时，setProperty 标准动作可以对一些常见数据类型直接转换，如字符串与 Integer 的转换就可以自动进行；
3. getProperty 动作：<jsp:getProperty name=”” property=””/>
    - getProperty 标准动作用来调用 JavaBean 对象的 getXxx 方法，将其返回值在当前位置输出。
    - name 是 JavaBean 对象的 id 值，property 的值是 getXxx 方法中的 Xxx 部分，首字母变小写。假设需要调用 getAddress 方法显示其返回值，那么 property 的值就是 Address 的首字母变小写，即 address。
