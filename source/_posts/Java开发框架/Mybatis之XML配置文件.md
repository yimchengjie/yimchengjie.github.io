---
title: Mybatis之XML配置文件
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Mybatis
date: 2018-04-22 15:38:49
toc: true
---

#### XML 配置文件

`mybatis-config.xml`配置文件包含了会影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

- configuration（配置）
  - properties（属性）
  - settings（设置）
  - typeAliases（类型别名）
  - typeHandlers（类型处理器）
  - objectFactory（对象工厂）
  - plugins（插件）
  - environments（环境配置）
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
  - databaseIdProvider（数据库厂商标识）
  - mappers（映射器）

#### properties（属性）

这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。

```java
//resource属性,映入java属性文件`config.properties`,也可以用url属性来指定路径
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

其中的属性就可以在整个配置文件中被用来替换需要动态配置,引用语法为`${property.name}`
也可以在 java 属性文件`config.properties`中来配置

如果属性在不只一个地方进行了配置，那么 MyBatis 将按照下面的顺序来加载：

1. 在 properties 元素体内指定的属性首先被读取。
2. 然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据 url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
3. 最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。

因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性。

#### settings（设置）

一个配置完整的 settings 元素的示例如下：

```html
<settings>
  <setting name="cacheEnabled" value="true" />
  <!-- 全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。 -->
  <setting name="lazyLoadingEnabled" value="true" />
  <!-- 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态。 -->
  <setting name="multipleResultSetsEnabled" value="true" />
  <!-- 是否允许单一语句返回多结果集（需要驱动支持）。 -->
  <setting name="useColumnLabel" value="true" />
  <!-- 使用列标签代替列名。不同的驱动在这方面会有不同的表现，具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。 -->
  <setting name="useGeneratedKeys" value="false" />
  <!-- 允许 JDBC 支持自动生成主键，需要驱动支持。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能支持但仍可正常工作（比如 Derby）。 -->
  <setting name="autoMappingBehavior" value="PARTIAL" />
  <!-- 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。 -->
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING" />
  <!-- 指定发现自动映射目标未知列（或者未知属性类型）的行为。
  NONE: 不做任何反应
  WARNING: 输出提醒日志 ('org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN)
  FAILING: 映射失败 (抛出 SqlSessionException) -->
  <setting name="defaultExecutorType" value="SIMPLE" />
  <!-- 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。 -->
  <setting name="defaultStatementTimeout" value="25" />
  <!-- 设置超时时间，它决定驱动等待数据库响应的秒数。 -->
  <setting name="defaultFetchSize" value="100" />
  <!-- 为驱动的结果集获取数量（fetchSize）设置一个提示值。此参数只可以在查询设置中被覆盖。 -->
  <setting name="safeRowBoundsEnabled" value="false" />
  <!-- 允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。	 -->
  <setting name="mapUnderscoreToCamelCase" value="false" />
  <!-- 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。 -->
  <setting name="localCacheScope" value="SESSION" />
  <!-- MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。 -->
  <setting name="jdbcTypeForNull" value="OTHER" />
  <!-- MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。 -->
  <setting
    name="lazyLoadTriggerMethods"
    value="equals,clone,hashCode,toString"
  />
  <!-- 	指定哪个对象的方法触发一次延迟加载。 -->
</settings>
```

#### typeAliases（类型别名）

类型别名是为 Java 类型设置一个短的名字。 它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。
例如：

```html
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author" />
  <typeAlias alias="Blog" type="domain.blog.Blog" />
  <typeAlias alias="Comment" type="domain.blog.Comment" />
  <typeAlias alias="Post" type="domain.blog.Post" />
  <typeAlias alias="Section" type="domain.blog.Section" />
  <typeAlias alias="Tag" type="domain.blog.Tag" />
</typeAliases>
```

当这样配置时，Blog 可以用在任何使用 domain.blog.Blog 的地方。

也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，
比如：

```html
<typeAliases>
  <package name="domain.blog" />
</typeAliases>
```

#### typeHandlers（类型处理器）

#### objectFactory（对象工厂）

#### plugins（插件）

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。

通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。

```html
<!-- mybatis-config.xml -->
<plugins>
  <plugin interceptor="org.mybatis.example.ExamplePlugin">
    <property name="someProperty" value="100" />
  </plugin>
</plugins>
```

#### environments（环境配置）

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中 使用相同的 SQL 映射。有许多类似的使用场景。
**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**
所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。

为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。例如

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, properties);
```

环境元素定义了如何配置环境。

```html
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..." />
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}" />
      <property name="url" value="${url}" />
      <property name="username" value="${username}" />
      <property name="password" value="${password}" />
    </dataSource>
  </environment>
</environments>
```

\*\*注意这里的关键点:

1. 默认使用的环境 ID（比如：default="development"）。
2. 每个 environment 元素定义的环境 ID（比如：id="development"）。
3. 事务管理器的配置（比如：type="JDBC"）。
4. 数据源的配置（比如：type="POOLED"）。\*\*

#### databaseIdProvider（数据库厂商标识）

#### mappers（映射器）

既然 MyBatis 的行为已经由上述元素配置完了，我们现在就要定义 SQL 映射语句了。 但是首先我们需要告诉 MyBatis 到哪里去找到这些语句。 Java 在自动查找这方面没有提供一个很好的方法，所以最佳的方式是告诉 MyBatis 到哪里去找映射文件。 你可以使用相对于类路径的资源引用， 或完全限定资源定位符（包括 file:/// 的 URL），或类名和包名等。例如：

```html
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml" />
  <mapper resource="org/mybatis/builder/BlogMapper.xml" />
  <mapper resource="org/mybatis/builder/PostMapper.xml" />
</mappers>

<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml" />
  <mapper url="file:///var/mappers/BlogMapper.xml" />
  <mapper url="file:///var/mappers/PostMapper.xml" />
</mappers>

<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper" />
  <mapper class="org.mybatis.builder.BlogMapper" />
  <mapper class="org.mybatis.builder.PostMapper" />
</mappers>

<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder" />
</mappers>
```
