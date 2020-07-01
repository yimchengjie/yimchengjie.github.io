---
title: Spring之容器和依赖注入
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Spring
toc: true
date: 2018-04-31 15:18:44
---
### Spring之容器和依赖注入

#### Bean容器

+ **BeanFactory**是基础的容器,由 org.springframework.beans.facytory.BeanFactory 接口定义, 负责创建各种Bean
创建BeanFactory实例时,需要提供Spring容器的详细配置文件`applicationContext.xml`

+ **ApplicationContext**是BeanFactory的子接口,它相比于BeanFactory,多了国际化支持等功能,而且在项目初始化过程中,ApplicationContext会自检,避免后续使用getBean方法时抛出异常
**通常使用`ApplicationContext`来创建Spring容器**

在Web项目中,Spring容器的实例化通常是由ContextLoaderListener来实现,只需要在web.xml中对其进行配置
例如:

```html
<!--指定Spring配置文件的位置，有多个配置文件时，以逗号分隔-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <!--spring将加载src/config目录下的applicationContext.xml文件-->
    <param-value>
        classpath:config/applicationContext.xml
    </param-value>
</context-param>
<!--指定以ContextLoaderListener方式启动Spring容器-->
<!-- 它在启动Web容器时，自动装配ApplicationContext的配置信息。 -->
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
<!-- 它实现了ServletContextListener接口，在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法。 -->
```

##### Bean管理

1. applicationContext.xml文件中`<bean>`标签对Bean进行配置

    ```html
    <bean id="UserDao" class="com.spring.dao.Impl.UserDaoImpl" />
    <bean id="UserService" class="com.spring.service.UserService">
      <property name="userDao" ref="userDao" />
    </bean>
    ```

2. 通过注释的方式配置Bean
`@Component`该注释用来描述Spring中的Bean,该注释是泛化概念.
`@Repository`用于数据访问层(Dao层)的类描述为Bean
`@Service`用于业务层(Service层)的类描述为Bean
`@Controller`用于描述控制层(Action层)的类为Bean
`@Autowired`用于描述Bean的依赖实例(例如Service中的属性Dao对象),当Dao只有一个实现类时,默认按照类型来匹配,当有多个实现类存在,就需要在相应的实现类上标注其名称
    例如:

      ```java
      @Repository("DaoImpl1")
      public class DaoImpl1 implements Dao{}

      @Repository("DaoImpl2")
      public class DaoImpl2 implements Dao{}

      @Service
      public class Service{
        @Autowired
        private Dao DaoImpl1; 自动装配的是DaoImpl1实现类
      }
      ```

    `@Resource`作用和`@Autowired`一样,区别在于它默认装配是按照名称,如果没有匹配再按照类型
    `@Qualifier`一般配合`@Autowired`使用

#### SpringIoC依赖注入

Spring 容器在创建被调用者的实例时，会自动将调用者需要的对象实例注入给调用者，这样，调用者通过 Spring 容器获得被调用者实例，这称为依赖注入。
例如:在Service中注入Dao接口的实现实例

##### 依赖注入的两种方式

1. 属性setter注入
  使用setter方法注入被依赖的实例,

    ```java
    public class UserService{
      private UserDao userDao;
      public void setUserDao(UserDao userDao){
        this.userDao=userDao;
      }
    }
    ```

    ```html
    <bean id="UserDao" class="com.spring.dao.Impl.UserDaoImpl" />
    <bean id="UserService" class="com.spring.service.UserService">
      <property name="userDao" ref="userDao" />
    </bean>
    ```

2. 构造方法注入(推荐这种方法)
  使用带参数的构造方法注入

    ```java
    public class UserService{
      private UserDao userDao;
      public UserService(UserDao userDao){
        this.userDao=userDao;
      }
    }
    ```

    ```html
    <bean id="UserDao" class="com.spring.dao.Impl.UserDaoImpl" />
    <bean id="UserService" class="com.spring.service.UserService">
      <constructor-arg name="userDao" ref="userDao" />
    </bean>
    ```

#### SpringIoC常用的注解

1. @Configuration 相当于xml配置中的`<beans>`
2. @Bean 相当于xml配置中的`<bean>`
   1. 定义initMethod和destroyMethod属性指定Bean的初始化和销毁方法@Bean(initMethod = "init",destroyMethod = "destroy")
3. @Scope("sigleton/prototype")配置作用域"单例/多实例", 默认为单例
4. @Lazy 懒加载,容器启动时不加载, 使用时才加载
5. @CompentScan 包扫描注解,在配置类上使用
6. @Import 直接导入组件,导入组件的id为全限定类名
7. @Conditionl(Condition.class),自定义一个Condition类实现Condition接口,重写matches方法限定条件

#### 后置处理器

##### BeanPostProcessor(Bean后置处理器)

在Bean初始化前后会回调BeanPostProcessor中的postProcessBeforeInitialization(初始前)和postProcessAfterInitialization(初始后)方法

可以自定义一个Bean后置处理器, 重写这两个方法, 方法中会传递进Bean实例和BeanName

##### BeanFactoryPostProcessor(Bean工厂的后置处理器)

触发时机是在 **Bean定义被注册进BeanFactory,Bean实例化之前**

提供postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)方法, 可以在Bean初始化后,实例化前执行

##### BeanDefinitionRegistryPostProcessor(Bean定义注册前置处理器)

继承了BeanFactoryPostProcessor,在Bean的定义注册之前触发

#### Aware接口

Aware接口是为了让Bean能够意识到Bean自身在Spring容器中的属性,比如实现了ApplicationContextAware接口的类,能够获得ApplicationContext,实现了BeanFactoryAware接口的类,能够获取BeanFactory对象

#### 自动装配

##### @Autowired

默认情况下, 按照类型进行装配, 如果有多个同类型组件, 那么就按照属性名来装配
在同类型组件中,给一个组件加上@Primary注解, 可以优先加载这个组件

配合@Qualifier("beanName")可以指定装配的组件

##### @Resource

按照类型装配,但是不支持@Primary和@Qualifier
