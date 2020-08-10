---
title: 深入理解SpringIoC
categories:
  - Java开发框架
tags:
  - Spring
  - Java开发框架
toc: true
date: 2019-07-10 16:30:45
---
## SpringIoC详解

### 容器

对于SpringIoC来说,最重要的就是容器了,容器管理这所有的Bean,控制这Bean的依赖注入

#### BeanFactory

早期基础容器, 可以理解成一个HashMap,key是BeanName,value是Bean实例.

BeanFactory作为最顶层的一个接口类，它定义了IoC容器的基本功能规范

#### ApplicationContext

应用上下午,高级容器,相比BeanFactory功能全面很多

#### BeanDefinition

Bean对象在Spring中是以BeanDefinition来描述的

Bean的解析主要就是对配置文件或者配置类的解析

### Bean的生命周期

1. Spring对bean进行实例化
2. Spring将值和bean的引用注入到bean对应的属性中
3. 如果bean实现了BeanNameAware接口，就将bean的ID传递给setBeanName（）方法
4. 如果bean实现了BeanFactoryAware接口，就调用setBeanFactory()方法，将BeanFactory容器实例注入
5. 如果bean实现了ApplicationContextAware接口，就调用setApplicationContext()方法，将bean所在的应用上下文的引用注入
6. 如果bean实现了BeanPostProcessor接口，就调用postProcessBeforeInitialization()方法
7. 如果bean实现了InitializingBean接口，就调用它们的afterPropertiesSet()方法，类似的如果bean使用了init-method声明初始化方法，该方法也会被调用
8. 如果bean实现了BeanPostProcessor接口，就调用它们的postProcessAfterInitialization()方法
9. 此时，bean准备就绪，已经可以被使用了，它们会一直驻留在应用上下文，知道应用上下文被销毁
10. 如果bean实现了DisposableBean接口，将调用destroy()方法，类似的如果bean使用了destory-method声明了销毁方法，该方法也会被调用

### SpringIoC注解驱动初始化过程

SpringIoC的初始化过程也是ApplicationContext容器的初始化过程

在Spring中管理注解Bean的容器实现类有`AnnotationConfigApplicationContext` 和 `AnnotationConfigWebApplicationContex`

这里以`AnnotationConfigApplicationContext`为例

入口:`AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);`

1. 调用AnnotationConfigApplicationContext构造函数

    ```java
    public AnnotationConfigApplicationContext(Class... annotatedClasses) {
        // 1. 先执行父类构造方法,再执行本类无参构造方法
        this();
        // 2. 注册带注解的类
        this.register(annotatedClasses);
        // 3. 更新容器
        this.refresh();
    }
    ```

2. this(); 默认先调用父类无参构造函数,构建初始对象**DefaultListableBeanFactory**,最基础的BeanFactory

   ```java
   public GenericApplicationContext() {
        this.customClassLoader = false;
        this.refreshed = new AtomicBoolean();
        this.beanFactory = new DefaultListableBeanFactory();
    }
   ```

3. 再调用当前类this();创建读取器和扫描器

   ```java
   public AnnotationConfigApplicationContext() {
        //BeanDefinition解析器; 用来解析带注解的bean
        this.reader = new AnnotatedBeanDefinitionReader(this);
        //ClassPath下的BeanDefinition的扫描器(用来扫描类)
        this.scanner = new ClassPathBeanDefinitionScanner(this);
    }
   ```

4. 创建注解模式下的BeanDefinition解析器AnnotatedBeanDefinitionReader

   ```java
   public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
        // BeanName产生器
        this.beanNameGenerator = new AnnotationBeanNameGenerator();
        // 作用域元数据解析器
        this.scopeMetadataResolver = new AnnotationScopeMetadataResolver();
        Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
        Assert.notNull(environment, "Environment must not be null");
        this.registry = registry;
        // @Conditionl条件表达式鉴别器
        this.conditionEvaluator = new ConditionEvaluator(registry, environment, (ResourceLoader)null);
        AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
    }
   ```

   跟进AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);

   ```java
   public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(BeanDefinitionRegistry registry, Object source) {
        DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
        if (beanFactory != null) {
            if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
                beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
            }

            if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
                beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
            }
        }

        Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet(4);
        RootBeanDefinition def;
        if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalConfigurationAnnotationProcessor")) {
            // 注册主配置类的后置处理器
            def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalConfigurationAnnotationProcessor"));
        }

        if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalAutowiredAnnotationProcessor")) {
            // 注册处理@Autowired注解的后置处理器
            def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalAutowiredAnnotationProcessor"));
        }

        if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalRequiredAnnotationProcessor")) {
            // 注册处理@Required注解的后置处理器
            def = new RootBeanDefinition(RequiredAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalRequiredAnnotationProcessor"));
        }

        if (jsr250Present && !registry.containsBeanDefinition("org.springframework.context.annotation.internalCommonAnnotationProcessor")) {
            // 注册处理JSR规范注解的后置处理器
            def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalCommonAnnotationProcessor"));
        }

        //注册处理jpa的后置处理器
        if (jpaPresent && !registry.containsBeanDefinition("org.springframework.context.annotation.internalPersistenceAnnotationProcessor")) {
            def = new RootBeanDefinition();

            try {
                def.setBeanClass(ClassUtils.forName("org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor", AnnotationConfigUtils.class.getClassLoader()));
            } catch (ClassNotFoundException var6) {
                throw new IllegalStateException("Cannot load optional framework class: org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor", var6);
            }

            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalPersistenceAnnotationProcessor"));
        }

        //注册处理事件监听方法的处理器
        if (!registry.containsBeanDefinition("org.springframework.context.event.internalEventListenerProcessor")) {
            def = new RootBeanDefinition(EventListenerMethodProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.event.internalEventListenerProcessor"));
        }

        //注册事件监听工厂
        if (!registry.containsBeanDefinition("org.springframework.context.event.internalEventListenerFactory")) {
            def = new RootBeanDefinition(DefaultEventListenerFactory.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.event.internalEventListenerFactory"));
        }

        //完成spring自身的后置处理器注册
        //到这一步, BeanDefinitionMap中已经保存了一些spring自带的后置处理器的定义信息了
        return beanDefs;
    }
   ```

5. 创建ClassPath下的BeanDefinition的扫描器ClassPathBeanDefinitionScanner

   ```java
   public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters, Environment environment, ResourceLoader resourceLoader) {
        this.beanDefinitionDefaults = new BeanDefinitionDefaults();
        this.beanNameGenerator = new AnnotationBeanNameGenerator();
        this.scopeMetadataResolver = new AnnotationScopeMetadataResolver();
        this.includeAnnotationConfig = true;
        Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
        //设置加载BeanDefinition的注册器
        this.registry = registry;
        //是否使用默认的过滤规则
        if (useDefaultFilters) {
            this.registerDefaultFilters();
        }
        //设置环境
        this.setEnvironment(environment);
        //设置资源加载器
        this.setResourceLoader(resourceLoader);
    }
   ```

   跟进this.registerDefaultFilters();

   ```java
   protected void registerDefaultFilters() {
        // 向includeFilters中添加所有@Component注解的类(其他的一些Bean注解也都有@Component)
        this.includeFilters.add(new AnnotationTypeFilter(Component.class));
        ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();

        try {
            this.includeFilters.add(new AnnotationTypeFilter(ClassUtils.forName("javax.annotation.ManagedBean", cl), false));
            this.logger.debug("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
        } catch (ClassNotFoundException var4) {
        }

        try {
            this.includeFilters.add(new AnnotationTypeFil        ter(ClassUtils.forName("javax.inject.Named", cl), false));
            this.logger.debug("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
        } catch (ClassNotFoundException var3) {
        }

    }
   ```

6. 注册Bean配置类register(annotatedClasses)

   ```java
   public void registerBean(Class<?> annotatedClass, String name, Class... qualifiers) {
        // 将Bean配置信息转换成AnnotatedGenericBeanDefinition  注解通用BeanDefinition
        AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);
        // 判断@Conditionl条件是否有跳过注册的
        if (!this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
            // 解析@Scope作用域, 没有则默认Singleton
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
            // 将作用域信息添加到BeanDefinition
            abd.setScope(scopeMetadata.getScopeName());
            // 设置beanName
            String beanName = name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry);
            AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
            // 解析@Qualifier
            if (qualifiers != null) {
                Class[] var7 = qualifiers;
                int var8 = qualifiers.length;

                for(int var9 = 0; var9 < var8; ++var9) {
                    Class<? extends Annotation> qualifier = var7[var9];
                    // 如果有@Primary注解,向BeanDefinition中写入首选bean
                    if (Primary.class == qualifier) {
                        abd.setPrimary(true);
                    // 如果有@Lazy注解, 设置懒加载
                    } else if (Lazy.class == qualifier) {
                        abd.setLazyInit(true);
                    } else {
                        abd.addQualifier(new AutowireCandidateQualifier(qualifier));
                    }
                }
            }

            // 封装一个BeanName和BeanDefinition之间的映射关系
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
            // 创建代理对象
            definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
            // 按BeanName将BeanDefinition注册到容器中
            BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
        }
    }
   ```

7. **refresh();  IoC容器启动的核心**

   ```java
   public void refresh() throws BeansException, IllegalStateException {
        synchronized(this.startupShutdownMonitor) {
            // 1. 刷新上下文之前的准备工作
            this.prepareRefresh();
            // 2. 获取初始化BeanFactory
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            // 3. 对BeanFactory进行属性填充
            this.prepareBeanFactory(beanFactory);

            try {
                // 4. 模板方法，注册自己添加的BeanPostFactoryProcessor
                this.postProcessBeanFactory(beanFactory);
                // 5. 执行BeanFactory后置处理器
                this.invokeBeanFactoryPostProcessors(beanFactory);
                // 6. 注册Bean后置注册器
                this.registerBeanPostProcessors(beanFactory);
                // 7. 初始化国际化资源处理器
                this.initMessageSource();
                // 8. 初始化应用事件多播器
                this.initApplicationEventMulticaster();
                // 9. 模板方法，调用某些特殊的bean的初始化，springboot中在这个地方启动tomcat
                this.onRefresh();
                // 10. 注册监听器到多播器上
                this.registerListeners();
                // 11. 实例化所有非懒加载的单例Bean
                this.finishBeanFactoryInitialization(beanFactory);
                // 12. 初始化容器生命周期事件处理器，并发布容器的生命周期事件
                this.finishRefresh();
            } catch (BeansException var9) {
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
                }

                this.destroyBeans();
                this.cancelRefresh(var9);
                throw var9;
            } finally {
                this.resetCommonCaches();
            }

        }
    }
   ```

   对代码进行进一步跟进,看看每一步中都做了什么:
    1. prepareRefresh()
       1. this.startupDate = System.currentTimeMillis();**设置启动时间**
       2. initPropertySources();**自定义属性设置**
       3. getEnvironment().validateRequiredProperties();**检验属性的合法性**
       4. earlyApplicationEvents = new LinkedHashSet();**创建早期应用事件集合**
    2. beanFactory = this.obtainFreshBeanFactory();
       1. this.beanFactory.setSerializationId(this.getId());**设置BeanFactoryID**
    3. prepareBeanFactory(beanFactory);
       1. addBeanPostProcessor(new ApplicationContextAwareProcessor(this));**添加一个ApplicationContextAwareProcessor**
       2. ignoreDependencyInterface(XXX.class); **设置忽略注入的接口实现类**
       3. registerResolvableDependency(XXX.class,beanFactory); **注册可解析的注入的组件**
       4. **添加编译时的AOP组件**
       5. **注册环境组件,系统属性组件,系统环境组件**
    4. postProcessBeanFactory(beanFactory);
       1. **子类重写这个方法,在BeanFactory准备完成创建之前做最后的步骤**
    5. invokeBeanFactoryPostProcessors(beanFactory);
       1. 执行BeanDefinitionRegistryPostProcessor
          1. 获取所有BeanDefinitionRegistryPostProcessor
          2. 先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor
          3. 再执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor
          4. 最后执行其他的BeanDefinitionRegistryPostProcessor
       2. 执行BeanFactoryPostProcessor
          1. 获取所有BeanFactoryPostProcessor
          2. 先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor
          3. 再执行实现了Ordered顺序接口的BeanFactoryPostProcessor
          4. 最后执行其他的BeanFactoryPostProcessor
    6. registerBeanPostProcessors(beanFactory);
       1. 获取所有的BeanPostProcessor
       2. 先BeanPostProcessor
       3. 注册MergedBeanDefinitionPostProcessor
       4. 最后创建一个ApplicationListenerDetector;检查是ApplicationListener的bean
    7. initMessageSource();
       1. 获取BeanFactory
       2. 判断容器中是否有MessageSource的组件
       3. 如果有则取用, 没有则创建DelegatingMessageSource
       4. 将国际化组件祖册到容器
    8. initApplicationEventMulticaster();
       1. 获取BeanFactory
       2. 判断容器是否有ApplicationEventMulticaster
       3. 如果有取用, 没有则创建SimpleApplicationEventMulticaster
       4. 将ApplicationEventMulticaster组件添加到BeanFactory
    9.  onRefresh();
       1.  留给子类,子类重写这个方法, 在容器刷新时可以自定义一些逻辑
    10. registerListeners();
        1.  获取容器中的ApplicationListener
        2.  将所有ApplicationListener添加到ApplicationEventMulticaster
        3.  派发早期事件earlyApplicationEvents
    11. finishBeanFactoryInitialization(beanFactory);
        1.  preInstantiateSingletons;初始化剩下的所有单实例Bean
            1.  获取容器中的单例beanName
            2.  如果beanName对应的bean不是抽象不是懒加载不是多实例的
            3.  getBean(beanName);
            4.  doGetBean(beanName);
            5.  标记要创建bean了,保证线程安全
            6.  getMergedLocalBeanDefinition(beanName);获得BeanDefinition
            7.  getDependsOn();获取依赖
            8.  递归getBean;创建所依赖Bean
            9.  将获取的依赖bean注册到denpendsOn集合
            10. createBean(beanName, ex1, args);创建bean
            11. doCreateBean(beanName, mbdToUse, args);
                1.  createBeanInstance(beanName, mbd, args);
                2.  调用前置处理器
                3.  属性赋值
                4.  执行初始化
                5.  执行后置处理器
                6.  注册bean销毁方法
                7.  添加到单例bean集合
    12. finishRefresh();
        1.  初始化生命周期有关后置处理器
        2.  执行容器刷新完成事件
        3.  将ApplicationContext注册到视图中

#### 总结

##### 大致流程

> 1. Spring容器在启动时,会先加载有关Bean定义信息的配置文件或者配置类(xml注册bean/注解注册bean)
> 2. BeanDefinitionReader将配置文件或者配置类解析成BeanDefinition,并存入容器中BeanDefinitionRegistry
> 3. Spring容器扫描BeanDefinitionRegistry中的所有BeanDefinition,使用BeanFactoryPostProcessor对它们进行加工, 主要是依赖处理和属性赋值
> 4. 实例化Bean时,封装Bean然后完成对Bean的属性设置工作
> 5. 利用Bean后置处理器,对完成的Bean进行加工

##### 重要组件

1. Resource
   xml、properties资源文件的抽象

2. ResourceLoader
   资源的加载, 解析xml、properties返回Resource

3. BeanDefinition
   保存了从配置文件中读取到的bean的各种信息,一个bean对应一个BeanDefinition
   有beanClass、scope、lazyInit等属性

4. BeanDefinitionReader
   定义读取组件，从Resource资源中读取出BeanDefinition

5. BeanDefinitionRegistry
   BeanFactory的实现类需要实现这个接口，所以所有BeanFactory都有注册BeanDefinition的功能
   其内部维护了一个Map，可以将BeanDefinition和beanName的对应关系添加进去

6. Enviroment
   环境，保存了程序运行的环境参数（JDK版本，jre等等）

7. BeanFactoryPostProcessor接口
   BeanFactory后置处理器, 扩展切口, 允许它的实现类在容器初始化前后进行相应操作
   典型的有PropertyPlaceholderConfigurer,占位符配置处理器

8. Aware接口
   对于实现了XXXXAware的bean,spring会注入相应的XXXX, 通过重写setXXXX的方法

9. BeanPostProcessor接口
   允许实现它的bean,在实例化前后做相应操作,在最前最后位置

10. InitializingBean接口
    允许实现它的bean,在实例化前后做相应操作, 在处理器before之后和after之前

11. DisposableBean接口
    允许实现它的bean,在摧毁前后做相应操作

12. FactoryBean接口
    允许实现它的bean,在beanFactory.getBean()获取该bean时, 会调用这个bean中重写的方法getObject,而不是直接返回该bean
    工厂模式的体现
