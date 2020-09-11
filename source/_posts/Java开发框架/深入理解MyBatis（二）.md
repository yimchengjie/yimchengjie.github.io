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
    typeAliasesElement(root.evalNode("typeAliases"));
    pluginElement(root.evalNode("plugins"));
    objectFactoryElement(root.evalNode("objectFactory"));
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    reflectionFactoryElement(root.evalNode("reflectionFactory"));
    settingsElement(settings);
    // read it after objectFactory and objectWrapperFactory issue #631
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    typeHandlerElement(root.evalNode("typeHandlers"));
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```