---
title: MySQL数据库之事务
categories:
  - 数据库
tags:
  - MySQL
  - Mybatis
toc: true
date: 2019-10-22 18:23:20
---
## MySQL数据库之事务

MySQL事务主要用于处理操作量大的一系列操作;比如在人员管理系统中,删除一个人员的同时还要删除人员的基本资料,也要删除一些其他相关的信息,这样的数据库操作就叫事务.

### MySQL事务处理的两种方式

1. 用BEGIN(开始一个事务),ROLLBACK(事务回滚),COMMIT(事务确认)实现
2. 用SET来改变MySQL的自动提交模式

### 实现事务管理

```java
// 模拟一次转账
@Test
public void test1() {
  Connection conn = null;
  PreparedStatement ps = null;
  try {
      conn = DBUtils.getConnection();
      //这里关闭自动事务提交, 相当于开启事务 begin ,并且需要手动提交
      conn.setAutoCommit(false);
      //用户A转出100
      ps = conn.prepareStatement("update account set money=money-100 where name='A'");
      ps.executeUpdate();
      //模拟异常
      int i = 10/0;
      //用户B转入100
      ps = conn.prepareStatement("update account set money=money+100 where name='B'");
      ps.executeUpdate();
      //提交事务
      conn.commit();
  } catch (Exception e) {
      try {
        //事务回滚
        conn.rollback();
      } catch (SQLException e1) {
        e1.printStackTrace();
      }
      e.printStackTrace();
  } finally {
      DBUtils.close(conn, ps, null);
  }
}
```

### Mybatis中的事务管理

Mybatis的事务在环境配置中, 环境配置传入SqlSessionFactory工厂

```xml
<environment>
  <!-- 使用JDBC开启事务 -->
  <transactionManager type="jdbc"/>
</environment>
```

开启事务后,Mybatis会创建一个TransactionFactory 事务工厂
Mybatis中Transaction(事务)是由TransactionFactory(事务工厂)创建的

事务由Connection实例管理

### Spring+Mybatis实现事务管理

使用Managed事务管理,Mybatis对事务的创建提交回滚没有实现,是交给容器来实现的

```xml
<!-- Mybatis配置 -->
<environment>
  <!-- Mybatis中开启Managed事务管理 -->
  <transactionManager type="managed"/>
</environment>

```html
<!-- Spring配置 -->
<tx:annotation-driven />
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource" />
</bean>
```

```java
//SpringBoot开启事务
@EnableTransactionManagement //开始事务
```

抽离出事务Service,使用@Transational注解事务方法

```java
@Service
public class TransationalService {

    @Autowired
    Service service

    @Transactional(rollbackFor = Exception.class)
    public void transationMethod() {
         // 调用其他增删改查, 组成一个事务
        service.select();
        service.update();
        service.delete();
    }
}
```
