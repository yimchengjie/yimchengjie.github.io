---
title: Spring之面向切面AOP
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Spring
toc: true
date: 2018-05-06 12:36:57
---

### Spring 之面向切面 AOP

AOP 是指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式

AOP 的全称是“Aspect Oriented Programming”，即面向切面编程，它将业务逻辑的各个部分进行隔离，使开发人员在编写业务逻辑时可以专心于核心业务，从而提高了开发效率。
其应用主要体现在事务处理、日志管理、权限控制、异常处理等方面。

#### AOP 基本概念

1. Aspect(切面): 通常是一个类，里面可以定义**切入点和通知**
2. JointPoint(连接点): 程序执行过程中明确的点，即调用的方法,方法名、当前传入的参数值等等都会封装在 JointPoint 的实例对象中,**一般作为参数代入到具体的切入方法中**
3. Advice(通知): AOP 在特定的切入点上执行增强处理，有 before,after,afterReturning,afterThrowing,around
4. Pointcut(切入点): 就是带有通知的连接点，在程序中主要体现为书写**切入点表达式,定义在方法上,添加要增强的一个或一组方法,该注释描述一个可重用的切入点,可以被 Adivce(通知)引用**
5. Proxy(代理): AOP 框架创建的对象，代理就是目标对象的加强。Spring 中的 AOP 代理可以使 JDK 动态代理，也可以是 CGLIB 代理，前者基于接口，后者基于子类

#### SpringAOP 应用实例

1. 编写业务代码

    ```java
    @Service
    public class PrintService{
      public void printStr(String str){
        System.out.println(str);
      }
    }
    ```

2. 定义一个切面类

    ```java
    @Component
    @Aspect
    public class PrintAspect{
      // 添加PrintService类中的public void printStr(String str)方法
      @Pointcut("execution(public void PrintService.printStr(String str))")
      public void printCut(){}

      @After("printCut()")
      public void printAfter(JoinPoint jp){
        String name = jp.getSignature().getName();
        System.out.println("After"+name);
      }

      @Before("printCut()")
      public void printBefore(JoinPoint jp){
        String name = jp.getSignature().getName();
        System.out.println("Before"+name);
      }
    }
    ```

3. 配置代理

    ```html
    <?xml version="1.0" encoding="UTF-8"?>
    <beans
      xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context-3.1.xsd"
    >
      <aop:aspectj-autoproxy />
      <!-- 启动aop代理 -->
      <context:component-scan base-package="cn.outofmemory" />
      <!-- 自动扫描包内的Bean -->
    </beans>
    ```

#### AOP 执行过程

##### 1. Spring 创建 IOC 容器

##### 2. 寻找切面类

Spring 在创建完对象后,寻找由@Aspect 修饰的切面类,并获得切面类中的方法

##### 3. 寻找切面类的方法中带有表达书的部分

找到方法注解@execution()中的内容

##### 4. 查找对应的方法

随后, Spring 检查所有的类, 并将上面找到的表达书与所有类比对,符合表达式的方法就是需要被代理的类

##### 5. 动态创建代理类

Spring 根据目标类创建代理类, 添加切面逻辑方法,并将代理类返回给 IOC 容器
