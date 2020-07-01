---
title: SpringBoot+Mybatis整合
categories:
  - Java开发框架
tags:
  - Java开发框架
  - SpringBoot
  - Mybatis
toc: true
date: 2018-7-12 14:25:16
---

### SpringBoot+MyBatis 整合

以往我们需要用到 MyBatis 时,需要繁琐的 xml 配置,而且很容易造成 spring 和 mybatis 的 jar 冲突, 非常麻烦
而在 SpringBoot 下,只需要添加 mybatis 的 starter 启动器,就能很方便的和 spring 整合了,无需 xml 配置

#### 1. 添加 mybatis 的 starter 包

```html
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>2.0.0</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.6</version>
</dependency>
```

#### 2. 在 application.properties 中添加数据源

```txt
mybatis.type-aliases-package=com.ycj.entity

spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/ssmdemo?characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=123456
```

springboot 会自动加载 application.properties 然后注入 DateSource 类中

#### 3. 在 springboot 启动类上添加 Mapper 包的扫描注解

```java
@SpringBootApplication
@MapperScan("com.ycj.mapper")
public class SpringBootApplicationMain
```

注意:也可以在每一个 mapper 接口上添加`@Mapper`注解, 不过比较麻烦,并且容易遗漏. 推荐直接扫描包

#### 4. 添加 mapper 接口,并用注解实现

```java
public interface UserMapper {
    @Select(" SELECT * FROM myuser WHERE uname Like CONCAT('%',#{uname},'%') ")
    List<Map<String,Object>> userInfo(User user);
}
```

#### 5. 完善代码,service 层和 controller 层

```java
@ResponseBody
@RequestMapping("/userInfo.do")
public List<Map<String, Object>> userInfo(User user){
    List<Map<String, Object>> list=userService.userInfo(user);
    return list;
}
```

#### 浏览器访问

![成功](success.png)

#### 错误记录

当mapper.xml放在src/main/java目录下是， 默认maven是不加载mapper.xml文件的

需要在pom.xml中配置加载xml文件

  ```html
  <build>
    <resources>
        <resource>
          <directory>src/main/java</directory>
          <includes>
            <include>**/*.xml</include>
          </includes>
        </resource>
      </resources>
  </build>
  ```

最好还是将mapper.xml文件放在Resources目录下
