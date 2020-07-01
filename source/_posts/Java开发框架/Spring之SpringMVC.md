---
title: Spring之SpringMVC
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Spring
  - SpringMVC
toc: true
date: 2018-05-29 11:35:25
---

### Spring 之 SpringMVC

Web 应用的分层通常是 MVC,对应着数据层-视图层-控制层
SpringMVC 就是控制层框架

#### SpringMVC 工作流程

Spring MVC 框架主要由 DispatcherServlet、处理器映射、控制器、视图解析器、视图组成

1. 客户端请求提交到 DispatcherServlet。
2. 由 DispatcherServlet 控制器寻找一个或多个 HandlerMapping，找到处理请求的 Controller。
3. DispatcherServlet 将请求提交到 Controller。
4. Controller 调用业务逻辑处理后返回 ModelAndView。
5. DispatcherServlet 寻找一个或多个 ViewResolver 视图解析器，找到 ModelAndView 指定的视图。
6. 视图(JSP 等视图层)负责将结果显示到客户端。

从宏观角度考虑，DispatcherServlet 是整个 Web 应用的控制器；从微观考虑，Controller 是单个 Http 请求处理过程中的控制器，而 ModelAndView 是 Http 请求过程中返回的模型（Model）和视图（View）。

#### SpringMVC 的实现实例

1. 在 web.xml 文件中配置部署 DispatcherServlet

   ```html
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns="http://java.sun.com/xml/ns/javaee"
     xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
     xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
     version="3.0"
   >
     <display-name>springMVC</display-name>
     <!-- 部署 DispatcherServlet -->
     <servlet>
       <servlet-name>springmvc</servlet-name>
       <servlet-class
         >org.springframework.web.servlet.DispatcherServlet
       </servlet-class>
       <!-- 加载springmvc-servlet.xml配置文件 -->
       <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:springmvc-servlet.xml</param-value>
       </init-param>
       <!-- 表示容器再启动时立即加载servlet -->
       <load-on-startup>1</load-on-startup>
     </servlet>
     <servlet-mapping>
       <servlet-name>springmvc</servlet-name>
       <!-- 处理所有URL -->
       <url-pattern>/</url-pattern>
     </servlet-mapping>
   </web-app>
   ```

2. 创建 Controller 类

   ```java
   public class LoginController implements Controller {
       public ModelAndView handleRequest(HttpServletRequest arg0,
               HttpServletResponse arg1) throws Exception {
           return new ModelAndView("/WEB-INF/jsp/register.jsp");
       }
   }
   ```

3. 创建 springmvc-servlet.xml 配置 Controller 的映射信息

   ```html
   <?xml version="1.0" encoding="UTF-8"?>
   <beans
     xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns:mvc="http://www.springframework.org/schema/mvc"
     xmlns:p="http://www.springframework.org/schema/p"
     xmlns:context="http://www.springframework.org/schema/context"
     xsi:schemaLocation="
             http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/context
             http://www.springframework.org/schema/context/spring-context.xsd
             http://www.springframework.org/schema/mvc
             http://www.springframework.org/schema/mvc/spring-mvc.xsd"
   >
     <!-- LoginController控制器类，映射到"/login" -->
     <bean name="/login" class="controller.LoginController" />
   </beans>
   ```

4. 在配置文件中定义视图解析器(ViewResolver)

   ```html
   <bean
     class="org.springframework.web.servlet.view.InternalResourceViewResolver"
   >
     <!--前缀-->
     <property name="prefix" value="/WEB-INF/jsp/" />
     <!--后缀-->
     <property name="suffix" value=".jsp" />
   </bean>
   ```

   上述视图解析器配置了前缀和后缀两个属性，因此 Controller 中只需要返回`login`,视图解析器就会自动添加前缀和后缀

#### 通过注释实现 SpringMVC

传统风格的控制器不仅需要在配置文件中部署映射，而且只能编写一个处理方法，不够灵活。
在基于注解的**控制器类中可以编写多个处理方法**，进而可以处理多个请求（动作），这就允许将相关的操作编写在同一个控制器类中，从而减少控制器类的数量，方便以后的维护。
基于注解的控制器**不需要在配置文件中部署映射**，仅需要使用 **RequestMapping** 注释类型注解一个方法进行请求处理。

注释实现的 Controller 类

```java
/**
* “@Controller”表示 IndexController 的实例是一个控制器
*
* @Controller相当于@Controller(@Controller) 或@Controller(value="@Controller")
*/
@Controller
@RequestMapping(value="/index")
public class IndexController {
    // 处理请求的方法
    @RequestMapping(value = "/login")
    public String login(HttpSession session,HttpServletRequest request){
        /**
         * login代表逻辑视图名称，需要根据Spring MVC配置
         * 文件中internalResourceViewResolver的前缀和后缀找到对应的物理视图
         */
        session.setAttribute("skey", "session范围的值");
        session.setAttribute("rkey", "request范围的值");
        return "login";
    }
    @RequestMapping("/register")
    public String register(Model model) {
        /*在视图中可以使用EL表达式${success}取出model中的值*/
        // model是一个包含 Map 的 Spring 框架类型。
        // 可以保存数据,然后在视图层用EL表达式提取
        model.addAttribute("success", "注册成功");
        return "register";
    }
}
```

使用注释需要在配置文件开启注释扫描

```html
<!-- 使用扫描机制扫描控制器类，控制器类都在controller包及其子包下 -->
<context:component-scan base-package="controller" />
```

#### SpringMVC 获取参数

1. 把**表单参数**写在控制器类相应方法的形参中，即形参名称与请求参数名称完全相同。适用于 get 和 post

    ```java
    @RequestMapping("/register")
    public String register(String uname,String upass,Model model) {
        if ("zhangsan".equals(uname)
                && "123456".equals(upass)) {
            logger.info("成功");
            return "login"; // 注册成功，跳转到 login.jsp
        } else {
            logger.info("失败");
            // 在register.jsp页面上可以使用EL表达式取出model的uname值
            model.addAttribute("uname", uname);
            return "register"; // 返回 register.jsp
        }
    }
    ```

2. 通过一个**实体 Bean** 来接收请求参数，Bean 的属性名称必须与请求参数名称相同.适用于 get 和 post

    ```java
    @RequestMapping("/login")
    public String login(UserForm user, HttpSession session, Model model) {
        if ("zhangsan".equals(user.getUname())
                && "123456".equals(user.getUpass())) {
            session.setAttribute("u", user);
            return "main"; // 登录成功，跳转到 main.jsp
        } else {
            model.addAttribute("messageError", "用户名或密码错误");
            return "login";
        }
    }
    ```

3. 通过 **HttpServletRequest** 接收请求参数适用于 get 和 post

    ```java
    @RequestMapping("/register")
    /**
    * 通过HttpServletRequest接收请求参数
    */
    public String register(HttpServletRequest request,Model model) {
        String uname = request.getParameter("uname");
        String upass = request.getParameter("upass");
        if ("zhangsan".equals(uname)
                && "123456".equals(upass)) {
            return "login"; // 注册成功，跳转到 login.jsp
        } else {
            // 在register.jsp页面上可以使用EL表达式取出model的uname值
            model.addAttribute("uname", uname);
            return "register"; // 返回 register.jsp
        }
    }
    ```

4. 通过 **@PathVariable** 获取 URL 中的参数,适用 get

    ```java
    public String register(@PathVariable String uname,@PathVariable String upass,Model model) {
        if ("zhangsan".equals(uname) && "123456".equals(upass)) {
            return "login"; // 注册成功，跳转到 login.jsp
        } else {
            // 在register.jsp页面上可以使用EL表达式取出model的uname值
            model.addAttribute("uname", uname);
            return "register"; // 返回 register.jsp
        }
    }
    ```

5. 通过 **@RequestParam** 接收请求参数,适用于 get 和 post

    ```java
    public String register(@RequestParam String uname,
        @RequestParam String upass, Model model) {
        if ("zhangsan".equals(uname) && "123456".equals(upass)) {
            logger.info("成功");
            return "login"; // 注册成功，跳转到 login.jsp
        } else {
            // 在register.jsp页面上可以使用EL表达式取出model的uname值
            model.addAttribute("uname", uname);
            return "register"; // 返回 register.jsp
        }
    }
    ```

6. 通过 **@ModelAttribute** 接收请求参数,适用于 get 和 post

    ```java
    @RequestMapping("/register")
    public String register(@ModelAttribute("user") UserForm user) {
        if ("zhangsan".equals(uname) && "123456".equals(upass)) {
            return "login"; // 注册成功，跳转到 login.jsp
        } else {
            // 使用@ModelAttribute("user")与model.addAttribute("user",user)的功能相同
            //register.jsp页面上可以使用EL表达式${user.uname}取出ModelAttribute的uname值
            return "register"; // 返回 register.jsp
        }
    }
    ```

#### SpringMVC 的转发和重定向

转发是服务器行为，重定向是客户端行为。
转发地址栏不变,请求数据不丢失,
重定向地址栏改变,请求数据丢失

在 Spring MVC 框架中，重定向与转发的示例代码如下：

```java
@RequestMapping("/login")
public String login() {
    //转发到一个请求方法（同一个控制器类可以省略/index/）
    return "forward:/index/isLogin";
}
@RequestMapping("/isLogin")
public String isLogin() {
    //重定向到一个请求方法
    return "redirect:/index/isRegister";
}
@RequestMapping("/isRegister")
public String isRegister() {
    //转发到一个视图
    return "register";
}
```

#### SpringMVC 的 JSON 数据交互

Spring MVC 提供了 MappingJackson2HttpMessageConverter 实现类默认处理 JSON 格式请求响应。该实现类利用 Jackson 开源包读写 JSON 数据，将 Java 对象转换为 JSON 对象和 XML 文档，同时也可以将 JSON 对象和 XML 文档转换为 Java 对象。

在使用注解开发时需要用到两个重要的 JSON 格式转换注解，分别是 @RequestBody 和 @ResponseBody。

- @RequestBody：用于将请求体中的数据绑定到方法的形参中，_该注解应用在方法的形参上_。**把传来的 JSON 数据转换为响应 java 对象**
- @ResponseBody：用于直接返回 return 对象，_该注解应用在方法上_。**表示该方法返回一个 JSON 格式数据**
