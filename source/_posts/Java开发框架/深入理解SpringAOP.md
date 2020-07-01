---
title: 深入理解SpringAOP
categories:
  - Java开发框架
tags:
  - Spring
  - Java开发框架
toc: true
date: 2019-07-21 21:30:45
---

## 深入理解 SpringAOP

AOP 是指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式

SpringAOP 的底层其实就是动态代理

### Spring 是如何使用动态代理的

1. 将业务组件和切面组件添加到容器中,
2. 创建对象的时候, 根据切入点表达式拦截的类, 加入通知,生成代理对象.
3. 如果目标对象有实现接口就用 JDK 代理, 反之就用 CGLIB 代理.

### SpringAOP 注解驱动原理

IoC容器启动中,创建了哪些和AOP有关的组件? 这些组件什么时候工作? 工作内容是什么?

#### 1. 实现入口@EnableAspectJAutoProxy

主要工作:

- @Import(AspectJAutoProxyRegistrar.class)给容器导入 `AspectJAutoProxyRegistrar` 组件(切面自动代理注册器)
- 利用 AspectJAutoProxyRegistrar 给容器中注册一个切面相关的 bean`AnnotationAwareAspectJAutoProxyCreator`(支持注解的 AspectJ 自动代理创建器)

```java
/*
 *代码跟进演示
 */
//出发点
@EnableAspectJAutoProxy
//跟进
@Import({AspectJAutoProxyRegistrar.class})
//点进AspectJAutoProxyRegistrar
AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
//点进registerAspectJAnnotationAutoProxyCreatorIfNecessary方法
return registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry, (Object)null);
//点进registerAspectJAnnotationAutoProxyCreatorIfNecessary
return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
//点进AnnotationAwareAspectJAutoProxyCreator
```

#### 2. AnnotationAwareAspectJAutoProxyCreator 创建过程

![AnnotationAwareAspectJAutoProxyCreator继承关系](/AnnotationAwareAspectJAutoProxyCreator继承关系.png)

主要关注的是它继承了BeanFactoryAware和BeanPostProcessor接口,

所以它的创建是在BeanFactory进行初始化,注册BeanPostProcessor阶段(也就是BeanFactoryPostProcessor注册完成之后)

registerBeanPostProcessors(beanFactory);注册 BeanPostProcessor 来处理拦截 bean 的创建(在bean创建前后执行)；

   1. 先获取 IOC 容器已经定义的需要创建对象的所有 BeanPostProcessor
   2. 注册 BeanPostProcessor,实际上就是创建 BeanPostProcessor 对象，保存在容器中；
      创建 internalAutoProxyCreator 的 BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
      1. 创建 Bean 的实例
      2. populateBean；给 bean 的各种属性赋值
      3. initializeBean
      4. invokeAwareMethods()：处理 Aware 接口的方法回调
      5. applyBeanPostProcessorsBeforeInitialization();应用后置处理器的 postProcessBeforeInitialization();
      6. invokeInitMethods()；执行自定义的初始化方法
      7. applyBeanPostProcessorsAfterInitialization()；执行后置处理器的 postProcessAfterInitialization();
      8. BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
   3. 把 BeanPostProcessor 注册到 BeanFactory 中；
      beanFactory.addBeanPostProcessor(postProcessor);

#### 3. 起作用阶段

BeanPostProcessor是在bean实例化前后起作用的, 所以它的执行是在beanFactroy对bean实例化的阶段进行

finishBeanFactoryInitialization(beanFactory);实例化所有单例 bean

1. 遍历所有Bean,依次创建对象,使用getBean(beanName);方法
2. 创建bean
    1. 先看看容器中是否存在bean,如果存在直接使用,没有再创建
    2. 创建bean;
        1. resolveBeforeInstantiation解析,如果能返回一个代理对象则直接用,不能继续
        2. createBean
            1. 拿到所有BeanPostProcessor并执行postProcessBeforeInstantiation
            2. doCreateBean真正的创建bean
            3. 创建完,执行

#### 4. AnnotationAwareAspectJAutoProxyCreator的作用

##### postProcessBeforeInstantiation

每一个bean创建之前,调用postProcessBeforeInstantiation

前提: 容器加载了@AspectJ注解的类,并加载了信息,再匹配切入点表达式与哪些类、方法匹配

1. 判断当前的bean是否在advisedBeans中(里面保存了所有需要增强的bean)
2. 判断当前bean是否Advice、Pointcut、Advisor、AopInfrastructureBean、被@Aspect注解的类型 或者是设置了跳过自动代理
3. 是否配置了跳过通知

##### postProcessAfterInitialization

前提：bean实例化完成

1. 获取bean的所有通知器
2. 找到当前bean所匹配的通知器
3. 对通知器进行优先级排序
4. 将通知器存入adviseBeans
5. 如果当前的bean需要注入切面,创建bean的代理对象(jdk代理或者cglib代理)
6. 向容器返回增强后的代理对象

#### 代理对象如何工作

容器中保存了基础组件的代理对象

1. 拦截器拦截目标方法的执行
2. 根据ProxyFactory对象获取将要执行的目标方法的增强器链
3. 以任务栏的形式执行通知和目标方法

### 总结

1. `@EnableAspectJAutoProxy` 开启AOP功能
   会在容器启动阶段注册一个组件 `AnnotationAwareAspectJAutoProxyCreator`
2. 容器开始创建
3. 容器注册后置处理器;
   `AnnotationAwareAspectJAutoProxyCreator` 继承了后置处理器接口. 被创建并注册进容器
4. 容器开始创建bean;
   1. 创建业务逻辑bean;和切面bean
   2. `AnnotationAwareAspectJAutoProxyCreator`拦截组件创建,如果有已代理对象,直接返回
   3. 组件创建
   4. `AnnotationAwareAspectJAutoProxyCreator`执行postProcessBeforeInstantiation,给需要增强bean创建代理对象
      获取目标对象以及advisor(增强器,切面的通知方法) 创建代理
5. 执行目标方法
   1. CglibAopProxy.intercept();拦截
   2. 执行通知任务链和目标方法
