---
title: Spring之事务管理
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Spring
toc: true
date: 2018-05-21 19:46:10
---

### Spring 之事务管理

什么是事务,就是一些列的步骤, 是一个整体,要么全都完成要么全都不完成.一个步骤失败,就要把已经完成的步骤全部回滚,回到没有运行之前.
比如:银行取钱,你账户金额减少->银行 ATM 出钱, 要么两个都要完成,要么两个都不完成
事务有四个特性:

1. 原子性:事务是一些列步骤的组成,是一个整体,不可以分割
2. 一致性:事务要么全部完成,要么全部不失败, 不能一部分完成一部分失败
3. 隔离性:
4. 持久性:一旦事务完成,结果不会受到影响,通常事务的结果会写入持久存储器

Spring 的事务管理, 是基于 AOP 面向切面编程来实现的.

#### Spring 事务管理的三个核心接口

1. **PlatformTransactionManager**
   该接口是一个事务管理器,用于管理事务,提供了三个事务操作方法:

   - TransactionStatus getTransaction（TransactionDefinition definition）：用于获取事务状态信息。
   - void commit（TransactionStatus status）：用于提交事务。
   - void rollback（TransactionStatus status）：用于回滚事务。
     在项目中，Spring 将 xml 中配置的事务详细信息封装到对象 `TransactionDefinition` 中，然后通过事务管理器的 getTransaction() 方法获得事务的状态（`TransactionStatus`），并对事务进行下一步的操作。

2. **TransactionDefinition**
   该接口是描述事务的对象,提供了五个事务相关信息的获取方法:

   - String getName()：获取事务对象名称。
   - int getIsolationLevel()：获取事务的隔离级别。
   - int getPropagationBehavior()：获取事务的传播行为。
     - 在事务管理过程中，传播行为可以控制是否需要创建事务以及如何创建事务。
   - int getTimeout()：获取事务的超时时间。
   - boolean isReadOnly()：获取事务是否只读。
     通常情况下，数据的查询不会改变原数据，所以不需要进行事务管理，而对于数据的增加、修改和删除等操作，必须进行事务管理。如果没有指定事务的传播行为，则 Spring3 默认的传播行为是 required。

3. TransactionStatus
   该接口是事务的状态,它描述事务在某一时间点上事务的状态信息.包含留个操作:

   - void flush() 刷新事务
   - boolean hasSavepoint() 获取是否存在保存点
   - boolean isCompleted() 获取事务是否完成
   - boolean isNewTransaction() 获取是否是新事务
   - boolean isRollbackOnly() 获取是否回滚
   - void setRollbackOnly() 设置事务回滚

#### 利用 XML 配置文件实现事务管理

1. 新建 Dao 层接口

   ```java
   public interface AccountDao {
       // 汇款
       public void out(String outUser, int money);
       // 收款
       public void in(String inUser, int money);
   }
   ```

2. 创建 Dao 的实现类 Impl

   ```java
   public class AccountDaoImpl implements AccountDao {
       // 注入JdbcTemplate,使用SpringJDBC访问数据库
       private JdbcTemplate jdbcTemplate;
       public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
           this.jdbcTemplate = jdbcTemplate;
       }
       // 汇款的实现方法
       public void out(String outUser, int money) {
           this.jdbcTemplate.update("update account set money =money-?"
                   + "where username =?", money, outUser);
       }
       // 收款的实现方法
       public void in(String inUser, int money) {
           this.jdbcTemplate.update("update account set money =money+?"
                   + "where username =?", money, inUser);
       }
   }
   ```

3. 创建 Service 以及其实现类

   ```java
   public interface AccountService {
       // 转账
       public void transfer(String outUser, String inUser, int money);
   }
   /*-----------------------------------------------------*/
   public class AccountServiceImpl implements AccountService{
       private AccountDao accountDao;
       public void setAccountDao(AccountDao accountDao) {
           this.accountDao = accountDao;
       }
       public void transfer(String outUser, String inUser, int money) {
           this.accountDao.out(outUser, money);
           this.accountDao.in(inUser, money);
       }
   }
   ```

4. 配置 Spring 的 XML 文件

   ```html
   <!-- 配置数据源，读取properties文件信息 -->
   <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
     <property name="driverClass" value="${jdbc.driverClass}" />
     <property name="jdbcUrl" value="${jdbc.jdbcUrl}" />
     <property name="user" value="${jdbc.user}" />
     <property name="password" value="${jdbc.password}" />
   </bean>
   <!-- 配置dao -->
   <bean id="accountDao" class="com.mengma.dao.impl.AccountDaoImpl">
     <property name="jdbcTemplate" ref="jdbcTemplate" />
   </bean>
   <!-- 配置service -->
   <bean id="accountService" class="com.mengma.service.impl.AccountServiceImpl">
     <property name="accountDao" ref="accountDao" />
   </bean>
   <!-- 事务管理器，依赖于数据源 -->
   <bean
     id="txManager"
     class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
   >
     <property name="dataSource" ref="dataSource" />
   </bean>
   <!-- 编写通知：对事务进行增强（通知），需要编写切入点和具体执行事务的细节 -->
   <tx:advice id="txAdvice" transaction-manager="txManager">
     <tx:attributes>
       <!-- 给切入点方法添加事务详情，name表示方法名称，*表示任意方法名称，propagation用于设置传播行为，read-only表示隔离级别，是否只读 -->
       <tx:method
         name="find*"
         propagation="SUPPORTS"
         rollback-for="Exception"
       />
       <tx:method
         name="*"
         propagation="REQUIRED"
         isolation="DEFAULT"
         read-only="false"
       />
     </tx:attributes>
   </tx:advice>
   <!-- aop编写，让Spring自动对目标生成代理，需要使用AspectJ的表达式 -->
   <aop:config>
     <!-- 切入点 -->
     <!-- AspectJ表达式,代表com.mengma.service包中所有的方法 -->
     <aop:pointcut
       expression="execution(* com.mengma.service.*.*(..))"
       id="txPointCut"
     />
     <!-- 切面：将切入点与通知整合 -->
     <aop:advisor pointcut-ref="txPointCut" advice-ref="txAdvice" />
   </aop:config>
   ```

5. 修改业务代码(模拟意外,比如断电)

   ```java
   public void transfer(String outUser, String inUser, int money) {
       this.accountDao.out(outUser, money);
       //模拟断电
       int i = 1/0;
       this.accountDao.in(inUser, money);
   }
   ```

6. 测试

   ```java
   public class AccountTest {
       @Test
       public void test() {
           // 获得Spring容器，并操作
           String xmlPath = "applicationContext.xml";
           ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
           AccountService accountService = (AccountService)applicationContext.getBean("accountService");
           accountService.transfer("zhangsan", "lisi", 100);
       }
   }
   ```

#### 利用 Annotation 注释方式实现事务管理

省去 xml 中的编写通知和 AOP 编写

1. xml 中添加注册驱动

    ```html
    <tx:annotation-driven transaction-manager="txManager" />
    ```

    还有要注意, 使用注释,需要在 xml 中开启注释处理器,指定扫描注释

2. 在业务代码(目标事务)中添加注释

    `@Transcational`

    ```java
    @Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.DEFAULT, readOnly = false)
    public class AccountServiceImpl {
        private AccountDao accountDao;
        public void setAccountDao(AccountDao accountDao) {
            this.accountDao = accountDao;
        }
        public void transfer(String outUser, String inUser, int money) {
            this.accountDao.out(outUser, money);
            // 模拟断电
            int i = 1 / 0;
            this.accountDao.in(inUser, money);
        }
    }
    ```
