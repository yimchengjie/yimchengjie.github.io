---
title: MySQL数据库之索引
categories:
  - 数据库
tags:
  - MySQL
toc: true
date: 2019-10-28 19:21:16
---
## MySQL数据库之索引

MySQL索引的简历对于MySQL的高效运行至关重要,索引可以大大提高MySQL检索速度

索引分单列索引和组合索引。单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。组合索引，即一个索引包含多个列。

实际上，索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录。所以建立索引会占用磁盘空间的索引文件。

对表中数据增删改时,索引也要进行相应维护

### 索引的使用

1. 在创建表的时候创建索引

    ```sql
    CREATE TABLE book(
        id INT NOT NULL PRIMARY KEY,
        name VARCHAR(50) NOT NULL,
        author VARCHAR(20) NOT NULL,
        info VARCHAR(255) NULL,
        INDEX(author)
    );
    ```

2. CREATE INDEX直接创建索引

    ```sql
    CREATE INDEX index_name ON table_name (column_list)
    CREATE UNIQUE INDEX index_name ON table_name (column_list)
    ```

3. 删除索引

    ```sql
    drop index index_name on table_name ;
    alter table table_name drop index index_name ;
    alter table table_name drop primary key ;
    ```

4. 组合索引和前缀索引

    ```sql
    -- 使用column_list1(4)建立column_list1的前4位的索引
    ALTER TABLE table_name ADD INDEX index_name (column_list1(4),column_list2,column_list3);
    ```

### 使用注意

#### 不走索引的情况

1. 索引列参与到计算式
2. 索引列参与到函数运算
3. 正则表达式匹配
4. 条件中带or
5. like '%XX%'
6. where中有不等号

注意: like'XXX%' 走索引

索引虽然大大提高了查询速度, 但是降低了更新速度;

### 索引技巧

1. 只要列中有NULL值, 索引就无效
2. 使用短索引,对长字段建索引时使用前缀建立
3. ORDER BY中的列不会使用索引
4. 使用like语句走索引要用'XXX%'
5. 不要在列上运算
6. 索引要建立在经常进行select操作的字段上
7. 索引要建立在值相对唯一的字段上
