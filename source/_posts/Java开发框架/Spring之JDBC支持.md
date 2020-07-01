---
title: Spring之JDBC支持
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Spring
toc: true
date: 2018-05-15 10:32:03
---

### Spring 之 JDBC 支持

Spring 框架针对数据库开发的应用提供了 JDBCTemplate 类,该类是 Spring 对 JDBC 支持的核心,提供了所有对数据库操作功能的支持

1. DataSource
   主要功能是获取数据库的连接,还可以引入数据库缓冲池和分布式事务的支持,可以作为访问数据库资源的标准接口
2. SQLExceptionTranslator
   该接口负责对 SQLException 进行转换,通过配置,可以使 JDBCTemplate 在需要处理 SQLException 时,完成一些转译工作

#### Spring 中 JDBC 的配置信息

在 Spring 配置文件中配置

```html
<?xml version="1.0" encoding="UTF-8"?>
<beans
  xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http:/www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd"
>
  <!-- 配置数据源 -->
  <bean
    id="dataSource"
    class="org.springframework.jdbc.dataSource.DriverManagerDataSource"
  >
    <!--数据库驱动-->
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <!--连接数据库的url-->
    <property name="url" value="jdbc:mysql://localhost/spring" />
    <!--连接数据库的用户名-->
    <property name="username" value="root" />
    <!--连接数据库的密码-->
    <property name="password" value="root" />
  </bean>
  <!--配置JDBC模板-->
  <bean id="jdbcTemplate" class="org.springframework.jdbc.core.jdbcTemplate">
    <!--默认必须使用数据源-->
    <property name="dataSource" ref="dataSource" />
  </bean>
  <!--配置注入类(Dao层)-->
  <bean id="xxx" class="xxx">
    <!-- 在Dao层中注入jdbcTemplate -->
    <property name="jdbcTemplate" ref="jdbcTemplate" />
  </bean>
  ...
</beans>
```

在 Dao 层中注入 jdbcTemplate 实例, 该实例提供了大量的查询和更新数据库的方法,比如`query()`,`update()`等
