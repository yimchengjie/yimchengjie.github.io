---
title: 深入理解MyBatis（一）
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Mybatis
date: 2020-09-05 14:38:38
toc: true
---
## 深入理解MyBatis（一）

MyBatis的整体架构分为三层，分别是基础支持层、核心接口层和接口层
![MyBatis整体架构](/MyBatis整体架构.jpg)

### 基础支持层

基础支持层位于MyBatis整体架构最底层，支撑者MyBatis的核心业务层，是整个框架的基石。

#### 解析器模块

解析起模块主要提供两个功能：一个是对Xpath进行封装，为MyBatis初始化时解析配置文件提供支持，另一个时为了处理SQL动态占位符提供支持。

![解析器模块](/解析器模块.png)

##### XPathParser(封装的XML解析器)

Mybatis提供的XPathParser封装了XPath（用来查询XMl文档的语言）、Document、EntityResolver（用于加载本地DTD文件，DTD就是标签的定义字典）

默认情况，解析mybatis-config.xml配置文件，会加载(http://mybatis.org/dtd/mybatis-3-config.dtd)这个DTD文档。

XPathParser中提供了一系列的eval方法，用于解析boolean、short、long、int、String、Node等类型的信息，更具路径和表达式找出节点或者属性，进行响应的类型转换

在XPathParser.evalString()中，调用了PropertyParser.parse()来处理节点中响应的默认值。
PropertyParser.parse()会创建GenericTokenParser解析器，并将默认值的处理委托给GenericTokenParser.parse()方法。

###### GenericTokenParser(通配符解析器)

在GenericTokenParser中标记了两个值，openToken和closeToken，记录了占位符的开始和结束。
GenericTokenParser.parse()找出占位符的字面值，并交给TokenHandler处理，返回解析后的结果，然后重新拼接字符串并返回。

###### TokenHandler(符号解析器)

TokenHandler几口定义了解析符号的方法。

在PropertyParser中使用了它的一个实现类 VariableTokenHandler来解析占位符。

GenericTokenParser仅仅是找到占位符，而具体的解析都是又TokenHandler的实现类来做的。

#### 反射模块

Mybatis在进行参数处理、结果映射的时候会设计大量的反射操作。为了简化反射操作的相关代码，Mybatis提供了反射模块，对常见的反射操作做了封装，提供简介的API。

##### Reflector&ReflectorFactory

Reflector是MyBatis中反射模块的基础，每一个Reflector对象对于一个类，在Reflector中存储了反射操作需要用到的类的元信息。

```java
// 对应的类
private Class<?> type;
// 可读熟悉的名称集合，可读熟悉就是存相应的getter方法的属性
private String[] readablePropertyNames = EMPTY_STRING_ARRAY;
// 可写属性的名称集合，可写属性就是存相应的setter方法的属性
private String[] writeablePropertyNames = EMPTY_STRING_ARRAY;
// 记录了属性的setter方法，key是属性名称，value是Invoker对象，Invoker封装了Method
private Map<String, Invoker> setMethods = new HashMap<String, Invoker>();
// 记录了相应的getter方法，key是属性名称，value是Invoker对象
private Map<String, Invoker> getMethods = new HashMap<String, Invoker>();
// 记录了属性的setter方法的参数值类型，key是属性名称，value是setter方法的参数类型
private Map<String, Class<?>> setTypes = new HashMap<String, Class<?>>();
// 记录了属性的getter方法的返回值类型，key是属性名称，value是getter方法的返回类型
private Map<String, Class<?>> getTypes = new HashMap<String, Class<?>>();
// 记录了默认的构造方法
private Constructor<?> defaultConstructor;
// 记录了所有属性名称的集合
private Map<String, String> caseInsensitivePropertyMap = new HashMap<String, String>();
```

ReflectorFactory接口实现了对Reflector的创建和缓存。
Default是一个具体实现类

```java
// 是否开启Reflector对象的缓存
private boolean classCacheEnabled = true;
// 使用ConcurrentMap实现对Reflector对象的缓存
private final ConcurrentMap<Class<?>, Reflector> reflectorMap = new ConcurrentHashMap<Class<?>, Reflector>();

// 为指定的Class创建Reflector对象，并将对象缓存到reflectorMap中
@Override
public Reflector findForClass(Class<?> type){
    if (classCacheEnabled) {
            // synchronized (type) removed see issue #461
        Reflector cached = reflectorMap.get(type);
        if (cached == null) {
        cached = new Reflector(type);
        reflectorMap.put(type, cached);
        }
        return cached;
    } else {
        return new Reflector(type);
    }
}
```

#### 类型转换模块

类型转换模块为简化配置文件提供了别名机制。另外，实现了JDBC类型与JAVA类型之间的转换，该功能在SQL语句绑定实参以及映射查询结果集时都会涉及。为SQL绑定参数是，为将Java数据转换为JDBC类型，而在结果映射时，会将数据转换为Java类型。

##### TypeHandler

MyBatis中所有的类型转换器都继承了TypeHandler接口，在TypeHandler定义了四个方法。

```java
public interface TypeHandler<T> {
    // 在通过PreparedStatement为SQL语句绑定参数时，会将数据由Java类型转换为JdbcType类型
  void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;

  // 从ResultSet中获取数据时会调用get方法，会将数据由JdbcType类型转换为Java类型
  T getResult(ResultSet rs, String columnName) throws SQLException;

  T getResult(ResultSet rs, int columnIndex) throws SQLException;

  T getResult(CallableStatement cs, int columnIndex) throws SQLException;

}
```

##### TypeAliasRegeistry

Mybatis可以为一个类添加一个别名，之后通过别名引用该类。它的结构比较简单，用`Map<String,Class<?>>`来管理别名与类型之间的关系

#### 日志模块

日志可以帮互助运维人员和管理人员快速找出系统的故障和瓶颈，也可以帮助开发人员与运维人员沟通。
在Java开发中常用的日志框架有Log4j、Log4j2、Appache Commons Log、java.util.logging、slf4j等。这些工具对外的接口不经相同，为了统一这些工具的接口，MyBatis定义了一套统一的日志接口供上层使用，并为上述常用的日志框架提供了相应的适配器。

#### 资源加载

资源加载模块主要是对类的加载起进行封装，确定类加载起的使用顺序，并且提供了加载类文件和其他资源文件的功能，

#### 数据源模块

在数据持久层中，数据源是一个非常重要的组建，其性能直接关系到整个数据持久层的性能。

#### 事务管理模块

控制数据库事务是一件非常重要的工作。MyBatis使用了Transaction接口对数据库事务进行了抽象。

#### binding模块

binding模块用来将用户自定义的Mapper几口与映射配置文件关联起来。

#### 缓存模块

MyBatis提供了两层缓存结构，使用Cache接口的实现。

##### Cache接口

```java
public interface Cache {

  /**
   * @return 该缓存对象的id
   */
  String getId();

  /**
   * 向缓存中添加数据，一般情况下，key是CacheKey，value是查询结果
   */
  void putObject(Object key, Object value);

  /**
   * 根据指定的key，在缓存中查找结果对象
   */
  Object getObject(Object key);

  /**
   * 删除key对于的缓存
   */
  Object removeObject(Object key);

  /**
   * 清空缓存
   */  
  void clear();

  /**
   * 缓存的个数，这个方法不会被Mybatis核心功能使用，所以可以提供空实现
   */
  int getSize();
  
  /** 
   * 获取读写锁，这个方法不会被Mybatis核心功能使用，所以可以提供空实现
   */
  ReadWriteLock getReadWriteLock();

}
```

##### CacheKey

由于MyBatis中涉及动态SQL等多方面因素，缓存Key不能仅仅通过一个String来表示，所以MyBatis通过CacheKey表示缓存key

```java
// 乘数，固定初始值质数37，不会变
private int multiplier;
// 当前hashCode值，初始值是质数17，
// 计算公式：hashcode=hashcode * multiplier  + object.hashCode()*count
private int hashcode;
// 所有更新对象的初始hashCode的和
private long checksum;
// 当前组合中对象的个数
private int count;
// 此List保存当前组合中所有的对象
private List<Object> updateList;
```
