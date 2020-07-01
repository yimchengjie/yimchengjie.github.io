---
title: SpringCloud入门
categories:
  - 分布式开发技术
tags:
  - Spring
  - 分布式开发技术
  - SpringCloud
toc: true
date: 2019-11-25 19:30:49
---
## SpringCloud入门

SpringCloud是一套用于分布式/微服务的框架

分布式简单的来说,就是将业务拆分,并部署到不同的服务器上

分布式系统面临的问题就叫CAP,即数据一致性,可用性,容错率

### SpringCloud的基础功能

+ 服务治理: SpringCloud Eureka
+ 客户端负载均衡: SpringCloud Ribbon
+ 服务容错保护: SpringCloud Hystrix
+ 声明式服务调用: SpringCloud Feign
+ API网关服务: SpringCloud ZuuI
+ 分布式配置中心: SpringCloud Config

### SpringCloud Eureka

分布式/微服务的系统,由于不同模块在物理层面被分开,运行在不同的服务器. 那么模块与模块之间的调用,或者说通讯(信息交互), 就需要用到**远程调用**
远程调用就会有很多的地址配置, 比如IP地址, 一旦IP地址发生改变, 维护就变得很麻烦

**服务治理框架**就是围绕着服务注册和服务发现机制来完成对微服务应用实例的自动化管理工具

系统服务器将自己的id和IP注册到Eureka服务上, 这样系统间调用就只需要通过id, 而IP则由Eureka维护

我们把**提供Eureka的服务器**称为**Eureka服务端**, 那些**将服务注册到Eureka服务端**的的服务器成为**Eureka客户端**

#### 1. Eureka Server的配置

Eureka Server可以看成是Zookeeper

```yml
register-with-eureka: false     #false表示不向注册中心注册自己。
fetch-registry: false     #false表示自己就是注册中心，不需要去检索服务
```

#### 2. Eureka Client的配置

Eureka Client分为服务提供者和服务消费者, 也可以既是提供者也是消费者

```yml
eureka:
  client:
    register-with-eureka: false  # 当前微服务不注册到eureka中(消费端) 如果是服务端就改为true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/  
```

#### 3. Eureka的治理机制

1. 服务提供者
   1. **服务注册**: 启动时发送请求将自己注册到Eureka Server上
   2. **服务续约**: 在注册完服务后,服务提供者会维护一个心跳来告诉Eureka Server服务状态可用
   3. **服务下线**: 当服务不可用时,会触发下线请求,通知Eureka Server
2. 服务消费者
   1. **获取服务**: 启动消费者时, 它会发生请求给Eureka Server, 获取上面注册的服务清单
   2. **服务调用**: 消费者获取服务清单以后,通过服务名调用具体的服务实例.在进行服务调用的时候,会优先访问同处于一个地区的服务提供方(同一服务多台服务器集群的情况)
3. Eureka Server
   1. **失效剔除**: 默认每隔一段时间将当前注册的服务清单中没有续约的服务剔除
   2. **自我保护**: Eureka Server在运行期间,会统计心跳失效的比例,如果一段时间内的心跳成功比例过低,,Eureka Server会将这个提供者的信息保存起来

#### 4. RestTemplate提交REST请求

RestTemplate是SpringCloud封装的工具类,实现获取具体服务实例的IP

```java
// 传统的方式，直接显示写死IP是不好的！
//private static final String REST_URL_PREFIX = "http://localhost:8001";

// 服务实例名, 这里只是名字,不是IP
private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";

/**
  * 使用 使用restTemplate访问restful接口非常的简单
  * (url, requestMap, ResponseBean.class)这三个参数分别代表 REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。
  */
@Autowired
private RestTemplate restTemplate;

@RequestMapping(value = "/consumer/dept/add")
public boolean add(Dept dept) {
    return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
}
```

### SpringCloud Ribbon

为了实现服务的高可用, 往往会使用集群的方式部署一个服务,而如何让集群合理分摊请求,就涉及到**负载均衡**

Ribbon是SpringCloud支持的**客户端的负载均衡**

通常的负载均衡都在服务端,比如Nginx

所谓的客户端负载均衡,就是由**获取了服务清单的消费者,来进行负载均衡算法的分配**

#### Ribbon的配置

Ribbon是支持负载均衡,默认的负载均衡策略是采用**轮询**的方式,也可以自定义

```java
@Configuration
public class MySelfRule
{
    @Bean
    public IRule myRule()
    {
        //return new RandomRule();// Ribbon默认是轮询，自定义为随机
        //return new RoundRobinRule();// Ribbon默认是轮询，自定义为随机
        return new RandomRule_ZY();// 自定义为每台机器5次
    }
}
```

自定义的方式很简单,继承**AbstractLoadBalancerRule**,重写`public Server choose(ILoadBalancer lb, Object key)`即可

### SpringCloud Hystrix

**Hystrix**是SpringCloud的容错机制

虽然实现了服务注册和发现,以及集群负载均衡

但是,如果有一个请求,需要调用多个服务,而有一个服务出现了问题,比如网络延迟,那会导致这个请求不可用, 如果是高并发式的请求, 那**所有的请求都会延迟甚至不可用**,进一步,请求的堆积,导致负载均衡饱和,集群资源耗尽,服务器瘫痪, 这就是"**雪崩**"

Hystrix为了防止这一系列问题,实现了**断路器**、**线程隔离**等一些系列保护机制

1. **断路器**

    **Fallback**(失败快速返回):当某个服务节点出现故障(类似电器短路), 断路器的故障监控就会发现他(类似保险丝),向调用方返回一个错误响应,而不是长久等待.这样就不会因为请求时间过长而占用资源,防止了进一步恶化

2. **线程隔离**

    为每个依赖服务设立单独的线程池,固定线程数量, 当请求过多(每个请求都会创建线程),线程池饱和就会拒绝请求. 这样就阻断了雪崩

#### Hystrix仪表盘

Hystrix仪表盘用于实时监控Hystrix的各项指标信息, 帮助我们发现存在的问题,从而采取相应措施

### SpringCloud Feign

Feign整合了Eurake和Hystrix, 使开发更加便捷

此外,还提供了**声明式服务调用**

#### 声明式服务调用

使用声明式服务调用可以实现使用HTTP请求远程服务能与调用本地方法一样的简单遍历

```java
// value --->指定调用哪个服务
// fallbackFactory--->熔断器的降级提示
@FeignClient(value = "MICROSERVICECLOUD-DEPT", fallbackFactory = DeptClientServiceFallbackFactory.class)
public interface DeptClientService {

    // 采用Feign我们可以使用SpringMVC的注解来对服务进行绑定！
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list();

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept);
}
```

```java
/**
 * Feign中使用断路器
 * 这里主要是处理异常出错的情况(降级/熔断时服务不可用，fallback就会找到这里来)
 */
@Component // 不要忘记添加，不要忘记添加
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept get(long id) {
                return new Dept().setDeptno(id).setDname("该ID：" + id + "没有没有对应的信息,Consumer客户端提供的降级信息,此刻服务Provider已经关闭")
                        .setDb_source("no this database in MySQL");
            }

            @Override
            public List<Dept> list() {
                return null;
            }

            @Override
            public boolean add(Dept dept) {
                return false;
            }
        };
    }
}
```

### SpringCloud Zuul

由于服务的增多,路由规则和实例维护都需要外部负载均衡(nigix)来做
而且为了保证所有服务的安全性,每一层都要设立验证

Zuul网关可以解决上述问题

#### 如何解决上述问题

1. Zuul通过与Eureka结合,将自身注册到Eureka,同时获得了所有注册服务的清单.外层的请求,都要经过Zuul,这就使维护服务实例的任务交给了服务治理框架自己完成

2. 在Zuul网关上进行统一调用验证服务,实现对微服务接口的拦截和校验

Zuul是整个系统对外暴露的唯一接口,解决的是外部请求调用的问题,Ribbo和Fegin虽然也有负载均衡的功能,但解决的是内部各个服务之间的调用问题

Zuul和Nginx并不冲突,可以同时使用

### SpringCloud Config

随着业务的扩展,服务越来越多, 每个服务都要存在配置文件, 而配置文件往往是修改频繁的地方.

Config可以解决分布式系统配置的管理问题,它包含Client和Server两部分

Config将所有的配置文件存放到统一的位置管理(Server端),其他的服务(Client)通过接口获取自己的配置

Config通常用Git来做Server端

Config实现了实时配置管理,结合 SpringCloud Bus,可以实现服务动态刷新配置(不需要重启)
