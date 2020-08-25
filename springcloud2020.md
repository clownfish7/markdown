[TOC]

### SpringCloud2020

#### 1. 微服务架构理论
&emsp; xxxxxxxxxxxxxxxxxxx

#### 2. 版本选择
##### 2.1 SpringBoot版本选择
> git源码地址：<https://github.com/spring-projects/spring-boot/releases/>  
> SpringBoot2.0新特性：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes

&emsp; 官方强烈建议升级到2.x以上版本
##### 2.2 SpringCloud版本选择
> git源码地址：<https://github.com/spring-projects/spring-cloud/wiki>  
> 官网：https://spring.io/projects/spring-cloud

##### 2.3 SpringCloud Alibaba版本选择
&emsp; xxxxxxxxxxxxxxxxxxx
##### 2.4 SpringBoot 和 SpringCloud 版本依赖关系
> <https://spring.io/projects/spring-cloud#overview>  
> <https://start.spring.io/actuator/info> 查看 json

&emsp;同时用boot和cloud，需要照顾cloud，由cloud决定boot版本

#### 3. Cloud 各组件停更/升级/替换
+ 服务注册中心
  + X Eureka
  + √ Zookeeper
  + √ Consul
  + √ Nacos
+ 服务调用
  + √ Ribbon
  + √ LoadBalancer
+ 服务调用2
  + X Feign
  + √ OpenFeign
+ 服务降级
  + X Hystrix
  + √ resilience4j
  + √ Sentinel
+ 服务网关
  + X Zuul
  + ! Zuul2
  + √ Gateway
+ 服务配置
  + X Config
  + √ Nacos
+ 服务总线
  + X Bus
  + √ Nacos

#### 4. 微服务架构编码构建
&emsp; 约定 > 配置 > 编码  
maven下载不了 -> <https://blog.csdn.net/HeyWeCome/article/details/104543411>

#### 5. Eureka 服务注册与发现
&emsp; Spring Cloud Eureka 是对Netflix公司的Eureka的二次封装，它实现了服务治理的功能，Spring Cloud Eureka提 供服务端与客户端，
服务端即是Eureka服务注册中心，客户端完成微服务向Eureka服务的注册与发现。服务端和 客户端均采用Java语言编写。
##### 5.1 Eureka 配置
&emsp; pom 引入依赖，2.x不同于1.x，显式区分了 client/server 
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```
&emsp; Springboot 启动类, zookeeper，consul等使用@EnableDiscoveryClient 
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(Eureka7001Application.class, args);
    }
}
```
&emsp; application.yml 配置文件
```yml
# 单机版
eureka:
  instance:
    #eureka服务端实例名称
    hostname: localhost
  client:
    #表示不向注册中心注册自己
    register-with-eureka: false
    #false表示自己就是注册中心，我的职责就是维护服务实例,并不需要去检索服务
    fetch-registry: false
    service-url:
      # 设置与eureka server交互的地址查询服务和注册服务都需要这个地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

# 集群版
eureka:
  instance:
    #eureka服务端实例名称
    hostname: 127.0.0.1
    #所在主机ip 都在自己机子上注意区分不同ip或者配置host，都填127.0.0.1我本机起来相互注册不上
    ip-address: 127.0.0.11
    #将自己的ip地址注册到Eureka服务中
    prefer-ip-address: true
    #指定实例id
    instance-id: ${spring.application.name}:${server.port}
    #eureka客户端向服务端发送心跳时间间隔，默认30s
    lease-renewal-interval-in-seconds: 30
    #eureka服务端在收到最后一次心跳后的等待时间，超时将删除服务，90s
    lease-expiration-duration-in-seconds: 90
  client:
    #表示是否向注册中心注册自己
    register-with-eureka: true
    #false表示自己就是注册中心，我的职责就是维护服务实例,并不需要去检索服务
    fetch-registry: false
    service-url:
      # 设置与eureka server交互的地址查询服务和注册服务都需要这个地址
      defaultZone: http://127.0.0.1:7002/eureka/
  server:
    #是否开启自我保护机制，默认为true
    enable-self-preservation: true
    #清理无效节点的时间间隔，默认60000毫秒，即60秒
    eviction-interval-timer-in-ms: 60000
```
##### 5.2 Eureka discoveryClient 服务发现
```java
@RestController
public class OrderController {
    @Autowired
    private DiscoveryClient discoveryClient;
    @GetMapping("/discoveryClient")
    public Object discoveryClient() {
        discoveryClient.getInstances("cloud-payment-service").forEach(System.out::println);
        return this.discoveryClient;
    }
}
//--- output ---
//org.springframework.cloud.netflix.eureka.EurekaDiscoveryClient$EurekaServiceInstance@4918212c
//org.springframework.cloud.netflix.eureka.EurekaDiscoveryClient$EurekaServiceInstance@47dfd67f
```
```json
{
  "discoveryClients": [
    {
      "services": [
        "cloud-payment-service",
        "cloud-consumer-order",
        "cloud-eureka-server"
      ],
      "order": 0
    },
    {
      "services": [
        
      ],
      "order": 0
    }
  ],
  "services": [
    "cloud-payment-service",
    "cloud-consumer-order",
    "cloud-eureka-server"
  ],
  "order": 0
}
```

##### 5.3 Eureka 自我保护机制
&emsp; 某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存,属于CAP里面的AP分支
&emsp; Eureka Server 在运行期间会去统计心跳失败比例在 15 分钟之内是否低于 85%，如果低于 85%，Eureka Server 会将这些实例保护起来，让这些实例不会过期，但是在保护期内如果服务刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，此时会调用失败，对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。
&emsp; 我们在单机测试的时候很容易满足心跳失败比例在 15 分钟之内低于 85%，这个时候就会触发 Eureka 的保护机制，一旦开启了保护机制，则服务注册中心维护的服务实例就不是那么准确了，此时我们可以使用eureka.server.enable-self-preservation=false来关闭保护机制，这样可以确保注册中心中不可用的实例被及时的剔除（不推荐）。
&emsp; 自我保护模式被激活的条件是：在 1 分钟后，Renews (last min) < Renews threshold。

+ Renews threshold ：Eureka Server 期望每分钟收到客户端实例续约的总数。
+ Renews (last min) ：Eureka Server 最后 1 分钟收到客户端实例续约的总数。 

解决方式有三种：
+ 关闭自我保护模式（eureka.server.enable-self-preservation设为false），不推荐。
+ 降低renewalPercentThreshold的比例（eureka.server.renewal-percent-threshold设置为0.5以下，比如0.49），不推荐。
+ 部署多个 Eureka Server 并开启其客户端行为（eureka.client.register-with-eureka不要设为false，默认为true），推荐。

&emsp; Eureka 的自我保护模式是有意义的，该模式被激活后，它不会从注册列表中剔除因长时间没收到心跳导致租期过期的服务，而是等待修复，直到心跳恢复正常之后，它自动退出自我保护模式。这种模式旨在避免因网络分区故障导致服务不可用的问题。例如，两个客户端实例 C1 和 C2 的连通性是良好的，但是由于网络故障，C2 未能及时向 Eureka 发送心跳续约，这时候 Eureka 不能简单的将 C2 从注册表中剔除。因为如果剔除了，C1 就无法从 Eureka 服务器中获取 C2 注册的服务，但是这时候 C2 服务是可用的。

&emsp; 所以，Eureka 的自我保护模式最好还是开启它。
#### 6. Zookeeper 服务注册与发现
&emsp;Eureka停止更新 <https://github.com/Netflix/eureka/wiki>   
zookeeper是一个分布式协调工具，可以实现注册中心功能，zookeeper服务器取代Eureka服务器，zk作为服务注册中心

##### 6.1 Zookeeper 服务提供者
&emsp; pom 引入依赖,注意 spring-cloud-starter-zookeeper-discovery 自带 zookeeper ，若与服务器上版本不一致则排除该 jar，引入对应版本
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud2020</artifactId>
        <groupId>com.clownfish7</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-provider-payment8004</artifactId>

    <dependencies>
        <!--SpringBoot整合Zookeeper客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <exclusions>
                <!--先排除自带的zookeeper3.5.3-->
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.6版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
        </dependency>
        <!--...-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency
    </dependencies>
</project>
```
&emsp; application.yml
```yml
server:
  port: 8004

spring:
  application:
    name: cloud-consumer-payment
  cloud:
    zookeeper:
      connect-string: 127.0.0.1:2181
```
&emsp; 正常编写 Controller

##### 6.2 Zookeeper 服务消费者
&emsp; pom 引入依赖,与上述服务提供者一致  
&emsp; application.yml，与上述服务提供者一致  
&emsp; 开启注解，注入 restTemplate，开启负载均衡  
```java
@SpringBootApplication
@EnableDiscoveryClient
public class Order80Application {
    public static void main(String[] args) {
        SpringApplication.run(Order80Application.class, args);
    }

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
&emsp; 调用服务提供者  
```java
@RestController
@RequestMapping("/payment")
public class OrderZkController {

    private static final String INVOKE_URL = "http://cloud-provider-payment";

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/payment/zk")
    public String get() {
        String result = restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
        return result;
    }

}
```

#### 7. Consul 服务注册与发现
&emsp; cocnsul 是 Go 语言编写的一款注册中心服务，<https://www.consul.io/intro/index.html>  
consul 提供以下功能：
+ 服务发现 提供 http 和 dns 两种发现方式
+ 健康检测 支持多种协议，http,tcp,docker,shell定制脚本
+ kv存储 key，value 的存储方式
+ 多数据中心 consul 支持多数据中心
+ 可视化 web 界面

下载地址： <https://www.consul.io/downloads.html>  
中文文档： <https://www.springcloud.cc/spring-cloud-consul.html>  
安装说明： <https://learn.hashicorp.com/consul/getting-started/install.html>  
访问地址： <http://localhost:8500>  

##### 7.1 Consul 服务提供者
&emsp; pom 引入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud2020</artifactId>
        <groupId>com.clownfish7</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-provider-payment8004</artifactId>

    <dependencies>
        <!--SpringBoot整合consul客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!--...-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency
    </dependencies>
</project>
```
&emsp; application.yml
```yml
server:
  port: 8006

spring:
  application:
    name: cloud-provider-payment
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```
&emsp; 正常编写 Controller

##### 7.2 Consul 服务消费者
&emsp; pom 引入依赖,与上述服务提供者一致  
&emsp; application.yml，与上述服务提供者一致  
&emsp; 开启注解，注入 restTemplate，开启负载均衡  
```java
@SpringBootApplication
@EnableDiscoveryClient
public class Order80Application {
    public static void main(String[] args) {
        SpringApplication.run(Order80Application.class, args);
    }

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
&emsp; 调用服务提供者  
```java
@RestController
@RequestMapping("/payment")
public class OrderConsulController {

    private static final String INVOKE_URL = "http://cloud-provider-payment";

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/payment/consul")
    public String get() {
        String result = restTemplate.getForObject(INVOKE_URL + "/payment/consul", String.class);
        return result;
    }

}
```

#### Eureka Zookeeper Consul 异同点
| 组件 | 语言 | CAP | 服务健康检查 | 对外暴露接口 | SpringCloud 集成 |
| --- | --- | --- | --- | --- | --- | 
| Eureka | Java | AP | 可配 | HTTP | 已集成 |
| Zookeeper | Java | CP | 支持 | 客户端 | 已集成 |
| Consul | Go | CP | 支持 | HTTP/DNS | 已集成 |

#### 8. Ribbon 负载均衡服务调用
&emsp; Ribbon是Netflix公司开源的一个负载均衡的项目（<https://github.com/Netflix/ribbon>），它是一个基于HTTP、 TCP的客户端负载均衡器。
  
1. 什么是负载均衡？ 负载均衡是微服务架构中必须使用的技术，通过负载均衡来实现系统的高可用、集群扩容等功能。负载均衡可通过
硬件设备及软件来实现，硬件比如：F5、Array等，软件比如：LVS、Nginx等。  
用户请求先到达负载均衡器（也相当于一个服务），负载均衡器根据负载均衡算法将请求转发到微服务。
负载均衡算法有：轮训、随机、加权轮训、加权随机、地址哈希等方法，负载均衡器维护一份服务列表，
根据负载均衡算法 将请求转发到相应的微服务上，所以负载均衡可以为微服务集群分担请求，降低系统的压力。

2. 什么是客户端负载均衡？ 客户端负载均衡与服务端负载均衡的区别在于客户端要维护一份服务列表，
Ribbon从 Eureka Server获取服务列表，Ribbon根据负载均衡算法直接请求到具体的微服务，中间省去了负载均衡服务。

&emsp; Spring Cloud引入Ribbon配合 restTemplate 实现客户端负载均衡。Java中远程调用的技术有很多，
如： webservice、socket、rmi、Apache HttpClient、OkHttp等，互联网项目使用基于http的客户端较多。

&emsp; pom 引入依赖
```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
</dependency>
```

&emsp; application.yml 配置 ribbon ,默认超时时间是 1 秒
```yml
ribbon:
  MaxAutoRetries: 2 #最大重试次数，当Eureka中可以找到服务，但是服务连不上时将会重试
  MaxAutoRetriesNextServer: 3 #切换实例的重试次数
  OkToRetryOnAllOperations: false #对所有操作请求都进行重试，如果是get则可以，如果是post，put等操作没有实现幂等的情况下是很危险的,所以设置为false
  ConnectTimeout: 5000 #请求连接的超时时间
  ReadTimeout: 6000 #请求处理的超时时间
```

&emsp; 定义RestTemplate，使用@LoadBalanced注解
```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate(new OkHttp3ClientHttpRequestFactory());
}
```

&emsp; 测试代码
```java
@Test
public void testRibbon() {
    //服务id
    String serviceId = "xxxxx";
    for (int i = 0; i < 10; i++) {
        //通过服务id调用
        ResponseEntity<Map> forEntity = restTemplate.getForEntity("http://"+serviceId+"/page/get/5a754adf6abb500ad05688d9", Map.class);
        Map body = forEntity.getBody();
    }
}
```

#### 9. OpenFeign 服务接口调用
&emsp; Feign是Netflix公司开源的轻量级rest客户端，使用Feign可以非常方便的实现Http 客户端。Spring Cloud引入 Feign并且集成了Ribbon实现客户端负载均衡调用。  

Feign工作原理如下：
1. 启动类添加@EnableFeignClients注解，Spring会扫描标记了@FeignClient注解的接口，并生成此接口的代理对象
2. @FeignClient(value = "xxx")即指定了cms的服务名称，Feign会从注册中 心获取cms服务列表，并通过负载均衡算法进行服务调用。
3. 在接口方法 中使用注解@GetMapping("/page/get/{id}")，指定调用的url，Feign将根据url进行远程调用。 

Feign注意点
SpringCloud对Feign进行了增强兼容了SpringMVC的注解 ，我们在使用SpringMVC的注解时需要注意：
1. feignClient接口 有参数在参数必须加@PathVariable("XXX")和@RequestParam("XXX")
2. feignClient返回值为复杂对象时其类型必须有`无参构造函数`。

&emsp; openFeign 中内置了 ribbon ，负载均衡默认采用轮询算法，默认超时参数为 1 秒
org.springframework.cloud.netflix.ribbon.RibbonClientConfiguration
```java
public class RibbonClientConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public IClientConfig ribbonClientConfig() {
        DefaultClientConfigImpl config = new DefaultClientConfigImpl();
        config.loadProperties(this.name);
        config.set(CommonClientConfigKey.ConnectTimeout, 1000);
        config.set(CommonClientConfigKey.ReadTimeout, 1000);
        config.set(CommonClientConfigKey.GZipPayload, true);
        return config;
    }
}
```

```yaml
ribbon:
  #请求处理的超时时间
  ReadTimeout: 5000
  #请求连接的超时时间
  ConnectTimeout: 5000
  #最大重试次数，当Eureka中可以找到服务，但是服务连不上时将会重试，如果eureka中找不到服务则直接走断路器,不包括首次调用
  MaxAutoRetries: 2
  #切换实例的重试次数,不包括首次调用
  MaxAutoRetriesNextServer: 3
  #对所有操作请求都进行重试，如果是get则可以，如果是post，put等操作没有实现幂等的情况下是很危险的,所以设置为false
  OkToRetryOnAllOperations: false
feign:
  hystrix:
    enabled: false

# 一般情况下 都是 ribbon 的超时时间（<）hystrix的超时时间（因为涉及到ribbon的重试机制）
# timeoutInMilliseconds的配置时间为:(1+MaxAutoRetries+MaxAutoRetriesNextServer)*ReadTimeout
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 18000
        timeout:
          #开启hystrix,为false将超时控制交给ribbon
          enabled: true
```

#### 10. Hystrix 断路器
##### Hystrix介绍
&emsp; 在微服务场景中，通常会有很多层的服务调用。如果一个底层服务出现问题，故障会被向上传播给用户。我们需要一种机制，当底层服务不可用时，可以阻断故障的传播。这就是断路器的作用。他是系统服务稳定性的最后一重保障。
       在springcloud中断路器组件就是Hystrix。Hystrix也是Netflix套件的一部分。他的功能是，当对某个服务的调用在一定的时间内（默认10s），有超过一定次数（默认20次）并且失败率超过一定值（默认50%），该服务的断路器会打开。返回一个由开发者设定的fallback。
       fallback可以是另一个由Hystrix保护的服务调用，也可以是固定的值。fallback也可以设计成链式调用，先执行某些逻辑，再返回fallback。  

##### Hystrix作用

&emsp; xxxxxxxxxxxxxxxxxxx
&emsp; 默认配置：com.netflix.hystrix.HystrixCommandProperties 超时 1 秒会 fallback

![head](https://gitee.com/clownfish7/image/raw/master/head/head.jpg 'head')

##### 10.1 服务降级
##### 10.2 服务熔断
##### 10.3 服务限流
##### 10.4 Hystrix 全部配置一览
此部分内容，可参考官方文档：https://github.com/Netflix/Hystrix/wiki/Configuration#execution.isolation.strategy
```java
@HystrixCommand(fallbackMethod = "str_fallbackMethod",
    groupKey = "strGroupCommand",
    commandKey = "strCommand",
    threadPoolKey = "strThreadPool",

    commandProperties = {
        //设置执行隔离策略，THREAD 表示线程池   SEMAPHORE:信号量隔离    默认为THREAD线程池
        @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
        // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小(最大并发数)
        @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
        // 配置命令执行的超时时间
        @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
        // 是否启用超时时间
        @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
        // 执行超时的时候是否中断
        @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
        // 执行被取消的时候是否中断
        @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
        // 允许回调方法执行的最大并发数
        @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
        // 服务降级是否启用，是否执行回调函数
        @HystrixProperty(name = "fallback.enabled", value = "true"),
        // 设置断路器是否起作用。
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
        // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，
        // 如果滚动时间窗(默认10s)内仅收到了19个请求，及时这19个请求都失败了，断路也不会打开。
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
        // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过 circuitBreaker.requestVolumeThreshold 的情况下，
        // 如果错误请求数的百分比超过 50，就把断路器设置为"打开"状态，否则就设置为"关闭"状态
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
        // 该属性用来设置当断路器打开之后的休眠时间窗。休眠时间窗结束之后，会将断路器置为"半开"状态，
        // 尝试熔断的请求命令，如果依然失败就将断路器继续设置为"打开"状态，如果成功就设置为"关闭"状态
        @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
        // 断路器强制打开
        @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
        // 断路器强制关闭
        @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
        // 滚动时间窗设置，该时间用于断路器判断健康度时，需要收集信息的持续时间
        @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
        // 该属性用来设置滚动时间窗统计指标信息时，划分"桶"的数量，断路器在手机指标信息的时候会根据设置的时间窗长度拆分成多个"桶"来累计各度量值，每个
        // "桶"记录了一段时间内的采集指标。比如 10 秒内拆分成 10 个"桶'收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
        @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
        // 该属性用来设置对命令执行的延迟是否采用百分位数来跟踪和计算。如果设置为 false，name所有的概要统计都将返回-1
        @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
        // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒
        @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
        // 该属性用来设置百分位统计滚动窗口中使用 "桶" 的数量
        @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
        // 该属性用来设置在执行过程中每个"桶"中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数
        // 就从最初的位置开始重写。例如，将该值设置为100，滚动窗口为10秒，若在10秒内一个"桶"中发生了500次执行，
        // 那么该"桶"中只保留最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间
        @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
        // 该属性用来设置采集意向断路器状态的健康快照(请求的成功、错误百分比)的间隔等待时间
        @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
        // 是否开启请求缓存
        @HystrixProperty(name = "requestCache.enabled", value = "true"),
        // HystrixCommand 的执行和事件是否打印日志到 HystrixRequestLog 中
        @HystrixProperty(name = "requestLog.enabled", value = "true")
    },
    threadPoolProperties = {
        // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
        @HystrixProperty(name = "coreSize", value = "10"),
        // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，否则将使用 LinkedBlockingQueue 实现的队列
        @HystrixProperty(name = "maxQueueSize", value = "-1"),
        // 该参数用来为队列设置拒绝阈值。通过该参数，即使队列没有达到最大值也能拒绝请求。该参数主要是对 LinkedBlockingQueue 队列的补充，因为LinkedBlockingQueue
        // 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了
        @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5")
    }
)
```

#### 11. Zuul 网关
&emsp; xxxxxxxxxxxxxxxxxxx
&emsp; pom 引入依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

&emsp; yml 配置
```yml
server:
  port: 50201
  servlet:
    context-path: /api
spring:
  application:
    name: xc-govern-gateway
  redis:
    host: ${REDIS_HOST:192.168.116.151}
    port: ${REDIS_PORT:6379}
    timeout: 5000 #连接超时 毫秒
    jedis:
      pool:
        maxActive: 3
        maxIdle: 3
        minIdle: 1
        maxWait: -1 #连接池最大等行时间 -1没有限制
zuul:
  routes:
    manage-course:
      path: /course/**
      serviceId: xc-service-manage-course #微服务名称，网关会从eureka中获取该服务名称下的服务实例的地址
      # 例子：将请求转发到http://localhost:31200/course
      #url: http://www.baidu.com #也可指定url，此url也可以是外网地址\
      strip-prefix: false #true：代理转发时去掉前缀，false:代理转发时不去掉前缀
      sensitiveHeaders:  #默认zuul会屏蔽cookie，cookie不会传到下游服务，这里设置为空则取消默认的黑名单，如果设置了具体的头信息则不会传到下游服务
      #   ignoredHeaders: 默认为空表示不过虑任何头
    xc-service-learning:  #路由名称，名称任意，保持所有路由名称唯一
      path: /learning/**
      serviceId: xc-service-learning #指定服务id，从Eureka中找到服务的ip和端口
      strip-prefix: false
      sensitiveHeaders:
    manage-cms:
      path: /cms/**
      serviceId: xc-service-manage-cms
      strip-prefix: false
      sensitiveHeaders:
    manage-sys:
      path: /sys/**
      serviceId: xc-service-manage-cms
      strip-prefix: false
      sensitiveHeaders:
    service-ucenter:
      path: /ucenter/**
      serviceId: xc-service-ucenter
      sensitiveHeaders:
      strip-prefix: false
    xc-service-manage-order:
      path: /order/**
      serviceId: xc-service-manage-order
      sensitiveHeaders:
      strip-prefix: false
eureka:
  client:
    registerWithEureka: true #服务注册开关
    fetchRegistry: true #服务发现开关
    serviceUrl: #Eureka客户端与Eureka服务端进行交互的地址，多个中间用逗号分隔
      defaultZone: ${EUREKA_SERVER:http://localhost:50101/eureka/}
  instance:
    prefer-ip-address:  true  #将自己的ip地址注册到Eureka服务中
    ip-address: 127.0.0.1
    instance-id: ${spring.application.name}:${server.port} #指定实例id
ribbon:
  MaxAutoRetries: 2 #最大重试次数，当Eureka中可以找到服务，但是服务连不上时将会重试，如果eureka中找不到服务则直接走断路器
  MaxAutoRetriesNextServer: 3 #切换实例的重试次数
  OkToRetryOnAllOperations: false  #对所有操作请求都进行重试，如果是get则可以，如果是post，put等操作没有实现幂等的情况下是很危险的,所以设置为false
  ConnectTimeout: 5000  #请求连接的超时时间
  ReadTimeout: 6000 #请求处理的超时时间
```

&emsp; 开启注解
```java
@SpringBootApplication
@EnableZuulProxy//此工程是一个zuul网关
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

&emsp; zuul 过滤器
```java
@Component
public class LoginFilter extends ZuulFilter {

    @Autowired
    AuthService authService;

    //过虑器的类型
    @Override
    public String filterType() {
        /**
         pre：请求在被路由之前执行
         routing：在路由请求时调用
         post：在routing和errror过滤器之后调用
         error：处理请求时发生错误调用
         */
        return "pre";
    }

    //过虑器序号，越小越被优先执行
    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        //返回true表示要执行此过虑器
        return true;
    }

    //过虑器的内容
    //测试的需求：过虑所有请求，判断头部信息是否有Authorization，如果没有则拒绝访问，否则转发到微服务。
    @Override
    public Object run() throws ZuulException {
        RequestContext requestContext = RequestContext.getCurrentContext();
        //得到request
        HttpServletRequest request = requestContext.getRequest();
        //得到response
        HttpServletResponse response = requestContext.getResponse();
        //取cookie中的身份令牌
//        String tokenFromCookie = authService.getTokenFromCookie(request);
//        if(StringUtils.isEmpty(tokenFromCookie)){
//            //拒绝访问
//            access_denied();
//            return null;
//        }
        //从header中取jwt
        String jwtFromHeader = authService.getJwtFromHeader(request);
        if(StringUtils.isEmpty(jwtFromHeader)){
            //拒绝访问
            access_denied();
            return null;
        }
        //从redis取出jwt的过期时间
//        long expire = authService.getExpire(jwtFromHeader);
//        if(expire<0){
//            //拒绝访问
//            access_denied();
//            return null;
//        }

        return null;
    }

    //拒绝访问
    private void access_denied(){
        RequestContext requestContext = RequestContext.getCurrentContext();
        //得到response
        HttpServletResponse response = requestContext.getResponse();
        //拒绝访问
        requestContext.setSendZuulResponse(false);
        //设置响应代码
        requestContext.setResponseStatusCode(200);
        //构建响应的信息
        ResponseResult responseResult = new ResponseResult(CommonCode.UNAUTHENTICATED);
        //转成json
        String jsonString = JSON.toJSONString(responseResult);
        requestContext.setResponseBody(jsonString);
        //转成json，设置contentType
        response.setContentType("application/json;charset=utf-8");
    }
}
```
```java
@Service
public class AuthService {

    @Autowired
    StringRedisTemplate stringRedisTemplate;

    //从头取出jwt令牌
    public String getJwtFromHeader(HttpServletRequest request){
        //取出头信息
        String authorization = request.getHeader("Authorization");
        if(StringUtils.isEmpty(authorization)){
            return null;
        }
        if(!authorization.startsWith("Bearer ")){
            return null;
        }
        //取到jwt令牌
        String jwt = authorization.substring(7);
        return jwt;


    }
    //从cookie取出token
    //查询身份令牌
    public String getTokenFromCookie(HttpServletRequest request){
        Map<String, String> cookieMap = CookieUtil.readCookie(request, "uid");
        String access_token = cookieMap.get("uid");
        if(StringUtils.isEmpty(access_token)){
            return null;
        }
        return access_token;
    }

    //查询令牌的有效期
     public long getExpire(String access_token){
        //key
         String key = "user_token:"+access_token;
         Long expire = stringRedisTemplate.getExpire(key, TimeUnit.SECONDS);
         return expire;
     }
}
```


#### 12. Gateway 新一代网关
&emsp; xxxxxxxxxxxxxxxxxxx

#### 13. SpringCloud config 分布式配置中心
&emsp; xxxxxxxxxxxxxxxxxxx

#### 14. SpringCloud Bus 消息总线
&emsp; xxxxxxxxxxxxxxxxxxx

#### 15. SpringCloud Stream 消息驱动
&emsp; xxxxxxxxxxxxxxxxxxx

#### 16. SpringCloud Sleuth 分布式请求链路追踪
&emsp; xxxxxxxxxxxxxxxxxxx

#### 17. SpringCloud Alibaba 入门简介
&emsp; xxxxxxxxxxxxxxxxxxx

#### 18. SpringCloud Alibaba Nacos 服务注册和配置中心
&emsp; xxxxxxxxxxxxxxxxxxx

#### 19. SpringCloud Alibaba Sentinel 熔断与限流
&emsp; xxxxxxxxxxxxxxxxxxx

#### 20. SpringCloud Alibaba Seata 处理分布式事务
&emsp; xxxxxxxxxxxxxxxxxxx