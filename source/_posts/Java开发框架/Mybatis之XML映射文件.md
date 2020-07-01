---
title: Mybatis之XML映射文件
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Mybatis
date: 2018-04-22 17:04:03
toc: true
---

### Mybatis 之 XML 映射文件

MyBatis 的真正强大在于它的映射语句，这是它的魔力所在。由于它的异常强大，映射器的 XML 文件就显得相对简单。

SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

- cache – 对给定命名空间的缓存配置。
- cache-ref – 对其他命名空间缓存配置的引用。
- resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
- sql – 可被其他语句引用的可重用语句块。
- insert – 映射插入语句
- update – 映射更新语句
- delete – 映射删除语句
- select – 映射查询语句

#### sql – 可被其他语句引用的可重用语句块

这个元素可以被用来定义可重用的 SQL 代码段，这些 SQL 代码可以被包含在其他语句中。它可以（在加载的时候）被静态地设置参数。 在不同的包含语句中可以设置不同的值到参数占位符上。比如：

```html
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```

#### CRUD 语句

```html
<select
  id="findCustomerById"
  parameterType="Integer"
  resultType="com.itheima.po.Customer"
>
  SELECT * FROM t_customer WHERE id = #{id}
</select>
<insert
  id="addCustomer"
  parameterType="com.itheima.po.Customer"
  keyProperty="id"
  useGeneratedKeys="true"
>
  INSERT INTO t_customer(username,jobs,phone) VALUES (#{username}, #{jobs},
  #{phone})
</insert>
<!-- update和delete和insert实现非常接近 -->
```

#### insert 和 update 的子标签 selectKey

提供给你一个与数据库中自动生成主键类似的行为，同时保持了 Java 代码的简洁。

```html
<insert id="addCustomer" parameterType="com.itheima.po.Customer">
  <selectKey keyProperty="id" resultType="Integer" order="BEFORE">
    select if(max(id) is null, 1, max(id)+1) as new newId from t_customer
  </selectKey>
  INSERT INTO t_customer(id,username,jobs,phone) VALUES (#{id},#{username},
  #{jobs}, #{phone})
</insert>
```

#### 结果映射

resultMap 元素是 MyBatis 中最重要最强大的元素。它可以让你从 90% 的 JDBC ResultSets 数据提取代码中解放出来，并在一些情形下允许你进行一些 JDBC 不支持的操作。实际上，在为一些比如连接的复杂语句编写映射代码的时候，一份 resultMap 能够代替实现同等功能的长达数千行的代码。ResultMap 的设计思想是，对于简单的语句根本不需要配置显式的结果映射，而对于复杂一点的语句只需要描述它们的关系就行了。

```html
<!-- 普通键值对存储 -->
<!-- 没有显式指定 resultMap -->
<select id="selectUsers" resultType="map">
  select id, username, hashedPassword from some_table where id = #{id}
</select>
<!--  将规范的JavaBean 映射到 ResultSet -->
<select id="selectUsers" resultType="com.someapp.model.User">
  select id, username, hashedPassword from some_table where id = #{id}
</select>
<!-- 如果列名和属性名没有精确匹配(不规范的JavaBean)，可以在 SELECT 语句中对列使用别名 -->
<select id="selectUsers" resultType="com.someapp.model.User">
  select user_id as "id", user_name as "userName", hashed_password as
  "hashedPassword" from some_table where id = #{id}
</select>
```

#### 高级结果映射

我们希望每个数据库都具备良好的第三范式或 BCNF 范式，可惜它们不总都是这样。
比如，我们如何映射下面这个语句？

```html
<!-- 非常复杂的语句 -->
<select id="selectBlogDetails" resultMap="detailedBlogResultMap">
  select B.id as blog_id, B.title as blog_title, B.author_id as blog_author_id,
  A.id as author_id, A.username as author_username, A.password as
  author_password, A.email as author_email, A.bio as author_bio,
  A.favourite_section as author_favourite_section, P.id as post_id, P.blog_id as
  post_blog_id, P.author_id as post_author_id, P.created_on as post_created_on,
  P.section as post_section, P.subject as post_subject, P.draft as draft, P.body
  as post_body, C.id as comment_id, C.post_id as comment_post_id, C.name as
  comment_name, C.comment as comment_text, T.id as tag_id, T.name as tag_name
  from Blog B left outer join Author A on B.author_id = A.id left outer join
  Post P on B.id = P.blog_id left outer join Comment C on P.id = C.post_id left
  outer join Post_Tag PT on PT.post_id = P.id left outer join Tag T on PT.tag_id
  = T.id where B.id = #{id}
</select>
```

你可能想把它映射到一个智能的对象模型，这个对象表示了一篇博客，它由某位作者所写，有很多的博文，每篇博文有零或多条的评论和标签。 我们来看看下面这个完整的例子，它是一个非常复杂的结果映射（假设作者，博客，博文，评论和标签都是类型别名）。

```html
<!-- 非常复杂的结果映射 -->
<resultMap id="detailedBlogResultMap" type="Blog">
  <constructor>
    <idArg column="blog_id" javaType="int"/>
  </constructor>
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
    <result property="favouriteSection" column="author_favourite_section"/>
  </association>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <association property="author" javaType="Author"/>
    <collection property="comments" ofType="Comment">
      <id property="id" column="comment_id"/>
    </collection>
    <collection property="tags" ofType="Tag" >
      <id property="id" column="tag_id"/>
    </collection>
    <discriminator javaType="int" column="draft">
      <case value="1" resultType="DraftPost"/>
    </discriminator>
  </collection>
</resultMap>
```

#### 缓存

默认情况下，只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。 要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：
`<cache/>`

#### 动态 SQL

如果你有使用 JDBC 或其它类似框架的经验，你就能体会到根据不同条件拼接 SQL 语句的痛苦。例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。

1. **if**
   动态 SQL 通常要做的事情是根据条件包含 where 子句的一部分。比如：

    ```html
    <select id="findActiveBlogWithTitleLike" resultType="Blog">
      SELECT * FROM BLOG WHERE state = ‘ACTIVE’
      <if test="title != null">
        AND title like #{title}
      </if>
    </select>
    ```

    这条语句提供了一种可选的查找文本功能。如果没有传入“title”，那么所有处于“ACTIVE”状态的 BLOG 都会返回；反之若传入了“title”，那么就会对“title”一列进行模糊查找并返回 BLOG 结果

2. **choose,when,otherwise**
   有时我们不想应用到所有的条件语句，而只想从中择其一项。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。

    ```html
    <select id="findActiveBlogLike" resultType="Blog">
      SELECT * FROM BLOG WHERE state = ‘ACTIVE’
      <choose>
        <when test="title != null">
          AND title like #{title}
        </when>
        <when test="author != null and author.name != null">
          AND author_name like #{author.name}
        </when>
        <otherwise>
          AND featured = 1
        </otherwise>
      </choose>
    </select>
    ```

3. **where**

    ```html
    <!-- 应对使用if时出现的语句错误 -->
    <!-- where 元素只会在至少有一个子元素的条件返回 SQL 子句的情况下才去插入“WHERE”子句。 -->
    <select id="findActiveBlogLike" resultType="Blog">
      SELECT * FROM BLOG
      <where>
        <if test="state != null">
          state = #{state}
        </if>
        <if test="title != null">
          AND title like #{title}
        </if>
        <if test="author != null and author.name != null">
          AND author_name like #{author.name}
        </if>
      </where>
    </select>
    ```

4. **foreach**

   动态 SQL 的另外一个常用的操作需求是对一个集合进行遍历，通常是在构建 IN 条件语句的时候。

    ```html
    <select id="selectPostIn" resultType="domain.blog.Post">
      SELECT * FROM POST P WHERE ID in
      <foreach
        item="item"
        index="index"
        collection="list"
        open="("
        separator=","
        close=")"
      >
        #{item}
      </foreach>
    </select>
    ```
