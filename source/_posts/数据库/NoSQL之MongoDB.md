---
title: NoSQL之MongoDB
categories:
  - 数据库
tags:
  - NoSQL
  - MongoDB
toc: true
date: 2020-01-03 17:21:16
---

## NoSQL 之 MongoDB

---

MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

### 安装 MongoDB

以 Ubuntu 为例

`sudo apt install mongodb`--安装命令

`systemctl status mongodb`--检查服务

### 概念解析

| SQL 术语    | MongoDB 术语 | 说明                                   |
| ----------- | ------------ | -------------------------------------- |
| database    | database     | 数据库                                 |
| table       | collection   | 表/集合                                |
| row         | document     | 记录行/文档                            |
| column      | field        | 字段/域                                |
| index       | index        | 索引                                   |
| table joins | -            | 连表查询/MongoDB 不支持                |
| primary key | primary key  | 主键,MongoDB 自动将\_id 字段设置为主键 |

### MongoDB Java

在 Java 中使用 MongoDB

直接使用 Spring Data MongoDB

1. 添加 spring-boot-starter-data-mongodb 包

   ```xml
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-mongodb</artifactId>
   </dependency>
   ```

2. 配置 application.properties

   ```java
   # user:用户名,
   # mongo1.example.com:host地址,
   # 27017:端口号
   # test:数据库名
   spring.data.mongodb.uri=mongodb://user:secret@mongo1.example.com:27017/test
   ```

3. 使用 MongoTemplate

MongoTemplate 提供了增删改查的方法

```java
@SpringBootTest
class SpiderApplicationTests {

    @Autowired
    private MongoTemplate mongoTemplate;

    @Test
    void contextLoads() {
        MongoCollection mongoCollection = mongoTemplate.getCollection("weibo");
        String index=mongoCollection.createIndex(new Document().append("text", 1));
        Long count=mongoCollection.countDocuments();
        mongoTemplate.createCollection("");
        System.out.println(index+"  "+count);
    }
}
```
