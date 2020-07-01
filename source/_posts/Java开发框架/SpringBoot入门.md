---
title: SpringBoot入门
categories:
  - Java开发框架
tags:
  - Java开发框架
  - SpringBoot
date: 2018-07-03 15:55:05
toc: true
---

### SpringBoot 简介

SpringBoot 是用来快速构建 Spring+SpringMvc 项目的框架,其设计就是为了尽可能的减少配置文件,快速开发 spring 应用

在使用 springboot 之前,我们开发一个 web 项目需要繁琐的配置, 还要担心 jar 的冲突和可行性

springboot 让一切都变得简单

- springboot 内嵌 Tomcat 等 web 容器
- 提供 starter 简化 maven 配置
- 自动配置 spring
- 无代码生成,无需 xml 配置
- 提供完善的项目测试
- 采用 java 类来配置

    **可以说 SpringBoot 是一个简便快捷强大的 Spring+SpringMvc,也是目前非常主流的 javaWeb 技术**

### 第一个 SpringBoot

#### 采用 maven 项目创建 springboot

1. 新建 maven 项目,在 pom.xml 导入 springboot 所需包

   ```html
   <parent>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-parent</artifactId>
     <version>2.0.4.RELEASE</version>
   </parent>

   <dependencies>
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
     </dependency>
   </dependencies>
   ```

2. 创建 springboot 启动入口
   该启动类位于 src 下,且与 controller,dao,service 等包同级

   ```java
   @SpringBootApplication
   public class SpringBootApplicationMain {

       public static void main(String[] args) {
           SpringApplication.run(SpringBootApplicationMain.class,args);
       }
   }
   ```

3. 创建一个 controller

   ```java
   @Controller
   @RequestMapping(value = "/user", produces="application/json;charset=UTF-8")
   @ResponseBody
   public class UserController {
       @RequestMapping("/hello.do")
       public String  hello(){
           return "Hello SpringBoot";
       }
   }
   ```

运行启动类, 无需 XML 配置, 无需 tomcat 配置, 创建快速,一键启动

1. 浏览器访问
   ![成功](success.png)

### SpringBoot 项目的基础依赖关系

**在项目目录使用命令:`mvn dependency:tree`**
![包依赖](包依赖.png)

### 对于 SpringBoot 中的 starter 的理解

starter 可以理解为启动器,它是一系列可以直接集成进 springboot 应用的依赖包的集合,只需要一站式的添加 starter 包,而不需要配置 XML,和解决包依赖、包冲突的问题。

### SpringBoot 接口开发

#### JSON 接口开发

在 controller 类上使用`@RestController`

或者在 controller 类的方法上加上`@ResponseBody`
