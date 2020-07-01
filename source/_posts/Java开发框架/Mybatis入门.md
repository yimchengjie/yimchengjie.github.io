---
title: Mybatis入门
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Mybatis
date: 2018-04-21 11:20:38
toc: true
---

#### 什么是 MyBatis

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

#### Mybatis 的优点

1. 简单小巧易于上手，方便浏览修改 SQL 语句
2. 解除 SQL 与程序代码的耦合
3. 提供映射标签，支持对象与数据库的 ORM 字段关系映射
4. 提供 xml 标签，支持编写动态 SQL

#### 安装

在 maven 项目中,导入以下

```html
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

#### 配置 SqlSessionFactory

每个基于 MyBatis 的应用都是以一个 `SqlSessionFactory` `的实例为核心的。SqlSessionFactory` 的实例可以通过 `SqlSessionFactoryBuilder` 获得。而 `SqlSessionFactoryBuilder` 则可以从 XML 配置文件或一个预先定制的 `Configuration` 的实例构建出 `SqlSessionFactory` 的实例。

```java
//引入mybatis-config.xml资源文件
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

mybatis-config.xml 文件包含了对 MyBatis 系统的核心设置，包含获取数据库连接实例的数据源（DataSource）和决定事务作用域和控制方式的事务管理器（TransactionManager）
简单实例:

```html
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC" />
      <dataSource type="POOLED">
        <property name="driver" value="${driver}" />
        <property name="url" value="${url}" />
        <property name="username" value="${username}" />
        <property name="password" value="${password}" />
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml" />
  </mappers>
</configuration>
```

要注意 XML 头部的声明，它用来验证 XML 文档正确性。environment 元素体中包含了事务管理和连接池的配置。mappers 元素则是包含一组映射器（mapper），这些映射器的 XML 映射文件包含了 SQL 代码和映射定义信息。

#### 从 SqlSessionFactory 中获取 SqlSession

既然有了 SqlSessionFactory，顾名思义，我们就可以从中获得 SqlSession 的实例了。

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
}
```

这里到底是怎么执行的?
先看看上述 mybatis-config.xml 文件中包含的映射(mapper)文件:`BlogMapper.xml`

```html
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

在命名空间 “org.mybatis.example.BlogMapper” 中定义了一个名为 “selectBlog” 的映射语句，允许你使用指定的完全限定名 “org.mybatis.example.BlogMapper.selectBlog” 来调用映射语句.这和使用完全限定名调用 Java 对象的方法类似。这样，该命名就可以直接映射到在命名空间中同名的 Mapper 类，并将已映射的 select 语句中的名字、参数和返回类型匹配成方法。 因此你就可以像上面那样很容易地调用这个对应 Mapper 接口的方法，就像下面这样：

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
```

#### 用注释来映射

对于像 BlogMapper 这样的映射器类来说，还有另一种方法来处理映射。 它们映射的语句可以不用 XML 来配置，而可以使用 Java 注解来配置。比如，上面的 XML 示例可被替换如下：

```java
package org.mybatis.example;
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}
```

使用注解来映射简单语句会使代码显得更加简洁，然而对于稍微复杂一点的语句，Java 注解就力不从心了，并且会显得更加混乱。 因此，**如果你需要完成很复杂的事情，那么最好使用 XML 来映射语句。**
