---
title: 深入理解MyBatis（二）
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Mybatis
date: 2020-09-08 20:01:38
toc: true
---
## 深入理解MyBatis（二）

--------

### 核心处理层

在核心处理层中，实现了包括MyBatis的初始化以及完成一次数据库操作的涉及的全部流程

#### MyBatis初始化

MyBatis的初始化主要工作是加载并解析mybatis-config.xml配置文件、映射配置文件以及相关的注解信息。

MyBatis的初始化入口是SqlSessionFactoryBuilder.build()方法，此方法会创建XMLConfigBuilder对象来解析mybatis-config.xml配置文件，得到Configuration对象，然后以此创建出DefaultSqlSessionFactory对象，完成初始化。

##### XMLConfigBuilder

XMLConfigBuilder负责解析mybatis-config.xml文件，核心如下

```java
// 用来标识是否解析过mybatis-config.xml文件
private boolean parsed;
// 用来解析配置文件的XPathParser
private XPathParser parser;
// 标识<environment>配置的名称，默认读区<environment>标签的default属性
private String environment;
// 负责创建和缓存Reflector对象
private ReflectorFactory localReflectorFactory = new DefaultReflectorFactory();

public Configuration parse() {
  // 根据parsed标记判断是否需要解析配置文件
  if (parsed) {
    throw new BuilderException("Each XMLConfigBuilder can only be used once.");
  }
  parsed = true;
  // 在配置文件中寻找<configuration>标签，并开始解析
  parseConfiguration(parser.evalNode("/configuration"));
  return configuration;
}

private void parseConfiguration(XNode root) {
  try {
    // 解析<settings>,settings下的属性会影响到MyBatis全局性配置，会改变MyBatis运行时行为。
    Properties settings = settingsAsPropertiess(root.evalNode("settings"));
    //issue #117 read properties first
    // 解析<properties>，在之后使用其中的内容替换占位符。
    propertiesElement(root.evalNode("properties"));
    loadCustomVfs(settings);
    // 类型别名
    typeAliasesElement(root.evalNode("typeAliases"));
    // 插件
    pluginElement(root.evalNode("plugins"));
    // 对象工厂
    objectFactoryElement(root.evalNode("objectFactory"));
    // 对象包装工厂
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    // 反射工厂
    reflectionFactoryElement(root.evalNode("reflectionFactory"));
    settingsElement(settings);
    // read it after objectFactory and objectWrapperFactory issue #631
    // 环境
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    // 类型处理器
    typeHandlerElement(root.evalNode("typeHandlers"));
    // 映射器
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```

##### XMLMapperBuilder

最重要的解析映射文件。程序入口如下：

```java
public void parse() {
  // 判断是否已经加载过该映射文件
  if (!configuration.isResourceLoaded(resource)) {
    // 处理mapper节点
    configurationElement(parser.evalNode("/mapper"));
    // 将resouce添加到已加载集合中
    configuration.addLoadedResource(resource);
    // 注册mapper接口
    bindMapperForNamespace();
  }
  // 处理上面方法中解析失败的<resultMap>节点
  parsePendingResultMaps();
  // 处理上面方法中解析失败的<cache-ref>节点
  parsePendingCacheRefs();
  // 处理上面方法中解析失败的SQL语句节点
  parsePendingStatements();
}
```

在`configurationElement(parser.evalNode("/mapper"))`方法中，有着各种节点的处理方法。

```java
private void configurationElement(XNode context) {
  try {
    // 获取mapper节点的namespace属性，为空时抛出异常
    String namespace = context.getStringAttribute("namespace");
    if (namespace == null || namespace.equals("")) {
      throw new BuilderException("Mapper's namespace cannot be empty");
    }
    builderAssistant.setCurrentNamespace(namespace);
    // 解析<cache-ref>节点
    cacheRefElement(context.evalNode("cache-ref"));
    // 解析<cache>节点
    cacheElement(context.evalNode("cache"));
    // 解析<parameterMap>节点（已被废弃）
    parameterMapElement(context.evalNodes("/mapper/parameterMap"));
    // 解析<resultMap>节点
    resultMapElements(context.evalNodes("/mapper/resultMap"));
    // 解析<sql>节点
    sqlElement(context.evalNodes("/mapper/sql"));
    // 解析sql语句
    buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
  }
}
```

当mapper文件解析注册完成以后， 整个MyBatis的初始化也就基本上差不多了。

##### MyBatis初始化过程总结

其实MyBatis的初始化说白了就是把配置文件和映射文件进行解析的过程。里面如何解析一个节点的代码其实不用特别care，因为那都是一些定义好的规则， 重要的时其中用到的思想，设计模式等。

整个MyBatis初始化看成一个方法，如参是配置文件映射文件，出参是DefaultSqlSessionFactory，而DefaultSqlSessionFactory中最重要的就是Configuration，xml文件解析出来的内容都被注册到了Configuration。然后Configuration影响后续MyBatis的具体行为。

#### 会话创建过程

我们每一次跟数据库通信，都要创建一个会话，我们使用openSession()方法创建

```java
public SqlSession openSession() {
  return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
}

private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
  Transaction tx = null;
  try {
    // 从配置中获取环境
    final Environment environment = configuration.getEnvironment();
    // 获取事务工厂类
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    // 创建事务
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
    // 创建执行器，以来执行SQL
    final Executor executor = configuration.newExecutor(tx, execType);
    // 默认的SqlSession
    return new DefaultSqlSession(configuration, executor, autoCommit);
  } catch (Exception e) {
    closeTransaction(tx); // may have fetched a connection so lets call close()
    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

#### SQL的生成、执行过程

回顾在JDBC中的过程
1. 获取Connection
2. 传入sql语句构建预编译对象
3. 设置参数
4. 执行SQL
5. 从结果集获取数据

在MyBatis中的过程，简单来说
1. 通过sqlSession，创建mapper代理类，执行调用SQL的方法，比如selectOne
2. 获取BoundSql对象，根据参数生成SQL语句
3. 从数据库连接池获取Connection，并创建代理
4. 从Connection中构建预编译对象
5. 设置参数
6. 执行SQL
7. 从结果集获取数据
8. 将数据转换为Java对象
