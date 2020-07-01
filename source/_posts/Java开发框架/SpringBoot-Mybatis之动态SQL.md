---
title: SpringBoot+Mybatis之动态SQL
categories:
  - Java开发框架
tags:
  - Java开发框架
  - SpringBoot
  - Mybatis
toc: true
date: 2018-07-25 16:28:07
---

### SpringBoot+Mybatis 之动态 SQL

在 ssm 项目中,使用 xml 文件利用标签,可以实现复杂的动态 sql 语句,那在 SpringBoot+Mybatis 项目下,注解的方式如何实现动态 sql

#### 1. 使用`<script>`标签包裹 XML 方式的语句

```java
public interface UserMapper {
    @Select(" <script> SELECT * FROM myuser " +
            " <where> " +
                " <if test='uid!=null' > " +
                    " uid=#{uid} " +
                " </if> " +
                " <if test='uname!=null' > " +
                   " and uname like CONCAT('%',#{uname},'%') " +
                " </if> " +
                " <if test='usex!=null' > " +
                  " and usex=#{usex} " +
                " </if> " +
            " </where> " +
            " </script> ")
    List<Map<String, Object>> userInfo(User user);
}
```

这个方式我们比较熟悉,但是用字符串包裹`<script>`语句就变得很复杂, 难以维护,所以不推荐, 这样的写法和 XML 的写法没有什么本质区别.

#### 2. 在 Mapper 中创建内部类,构建 sql

`@SelectProvider`等`@XXXProvider`注解允许我们指定一个类的方法来返回 sql 语句

```java
public interface UserMapper {
    @SelectProvider(type = UserMapperProvider.class , method = "userInfo")
    List<Map<String, Object>> userInfo(User user);

    class UserMapperProvider {
        public String userInfo(User user) {
            StringBuffer sql = new StringBuffer();
            sql.append(" SELECT * FROM myuser where 1=1 ");
            if (user.getUid() != null) {
                sql.append(" AND uid=#{uid} ");
            }
            if (user.getUname() != null) {
                sql.append(" AND uname like CONCAT('%',#{uname},'%') ");
            }
            if (user.getUsex() != null) {
                sql.append(" AND usex=#{usex} ");
            }
            return sql.toString();
        }
    }
}
```

#### 3. 创建内部类,使用结构化 SQL 方法

```java
public interface UserMapper {
    @SelectProvider(type = UserMapperProvider.class , method = "userInfo")
    List<Map<String, Object>> userInfo(User user);

    class UserMapperProvider {
        public String userInfo(User user) {
            return new SQL(){{
                SELECT(" * ");
                FROM(" myuser ");
                if (user.getUid() != null) {
                    WHERE(" uid=#{uid} ");
                }
                if (user.getUname() != null) {
                    WHERE(" uname like CONCAT('%',#{uname},'%') ");
                }
                if (user.getUsex() != null) {
                    WHERE(" AND usex=#{usex} ");
                }
            }
            }.toString();
        }
    }
}
```

##### 总结

> **方法 1 是 XML 语法的注解方式实现,方法 2 和方法 3 比较相似,可读性高,易于维护, 方法 2 适用面可能更广,而且更加接近我们熟悉的自然 sql 语法, 推荐使用方法 2**
