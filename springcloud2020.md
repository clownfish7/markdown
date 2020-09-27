[TOC]

### SpringCloud2020

#### 1. 微服务架构理论
> 什么是微服务架构：<https://www.zhihu.com/question/289101554/answer/734306979>  

  微服务是一种架构概念，旨在通过将功能分解到各个离散的服务中已实现对解决的解耦。你可以将其看作是在架构层次而非获取服务的类上应用很多 SOLID 原则。
微服务架构是一个很有趣的概念，它的主要作用是将功能分解到离散的各个服务中，从而降低系统的耦合性，并提供更加灵活的服务支持。
概念：把一个大型的单个应用程序和服务拆分为数个甚至数十个的支持服务，它可扩展单个组件而不是整个的应用程序堆栈，从而满足服务等级协议。
定义：围绕业务领域组件来创建应用，这些应用可以独立的进行开发，管理，和迭代。在分散的组件中使用云架构和平台式部署，管理和服务功能。使产品交付变得更加简单。
本质：用一些功能比较明确，业务比较精炼的服务区解决更大，更实际的问题

#### 2. 版本选择
##### 2.1 SpringBoot版本选择
> git源码地址：<https://github.com/spring-projects/spring-boot/releases/>  
> SpringBoot2.0新特性：https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes

  官方强烈建议升级到2.x以上版本
##### 2.2 SpringCloud版本选择
> git源码地址：<https://github.com/spring-projects/spring-cloud/wiki>  
> 官网：https://spring.io/projects/spring-cloud

##### 2.3 SpringCloud Alibaba版本选择
  xxxxxxxxxxxxxxxxxxx
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
  约定 > 配置 > 编码  
maven下载不了 -> <https://blog.csdn.net/HeyWeCome/article/details/104543411>

#### 5. Eureka 服务注册与发现
  Spring Cloud Eureka 是对Netflix公司的Eureka的二次封装，它实现了服务治理的功能，Spring Cloud Eureka提 供服务端与客户端，
服务端即是Eureka服务注册中心，客户端完成微服务向Eureka服务的注册与发现。服务端和 客户端均采用Java语言编写。
##### 5.1 Eureka 配置
  pom 引入依赖，2.x不同于1.x，显式区分了 client/server 
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```
  Springboot 启动类, zookeeper，consul等使用@EnableDiscoveryClient 
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(Eureka7001Application.class, args);
    }
}
```
  application.yml 配置文件
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
  某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存,属于CAP里面的AP分支
  Eureka Server 在运行期间会去统计心跳失败比例在 15 分钟之内是否低于 85%，如果低于 85%，Eureka Server 会将这些实例保护起来，让这些实例不会过期，但是在保护期内如果服务刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，此时会调用失败，对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。
  我们在单机测试的时候很容易满足心跳失败比例在 15 分钟之内低于 85%，这个时候就会触发 Eureka 的保护机制，一旦开启了保护机制，则服务注册中心维护的服务实例就不是那么准确了，此时我们可以使用eureka.server.enable-self-preservation=false来关闭保护机制，这样可以确保注册中心中不可用的实例被及时的剔除（不推荐）。
  自我保护模式被激活的条件是：在 1 分钟后，Renews (last min) < Renews threshold。

+ Renews threshold ：Eureka Server 期望每分钟收到客户端实例续约的总数。
+ Renews (last min) ：Eureka Server 最后 1 分钟收到客户端实例续约的总数。 

解决方式有三种：
+ 关闭自我保护模式（eureka.server.enable-self-preservation设为false），不推荐。
+ 降低renewalPercentThreshold的比例（eureka.server.renewal-percent-threshold设置为0.5以下，比如0.49），不推荐。
+ 部署多个 Eureka Server 并开启其客户端行为（eureka.client.register-with-eureka不要设为false，默认为true），推荐。

  Eureka 的自我保护模式是有意义的，该模式被激活后，它不会从注册列表中剔除因长时间没收到心跳导致租期过期的服务，而是等待修复，直到心跳恢复正常之后，它自动退出自我保护模式。这种模式旨在避免因网络分区故障导致服务不可用的问题。例如，两个客户端实例 C1 和 C2 的连通性是良好的，但是由于网络故障，C2 未能及时向 Eureka 发送心跳续约，这时候 Eureka 不能简单的将 C2 从注册表中剔除。因为如果剔除了，C1 就无法从 Eureka 服务器中获取 C2 注册的服务，但是这时候 C2 服务是可用的。

  所以，Eureka 的自我保护模式最好还是开启它。
#### 6. Zookeeper 服务注册与发现
&emsp;Eureka停止更新 <https://github.com/Netflix/eureka/wiki>   
zookeeper是一个分布式协调工具，可以实现注册中心功能，zookeeper服务器取代Eureka服务器，zk作为服务注册中心

##### 6.1 Zookeeper 服务提供者
  pom 引入依赖,注意 spring-cloud-starter-zookeeper-discovery 自带 zookeeper ，若与服务器上版本不一致则排除该 jar，引入对应版本
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
        </dependency>
    </dependencies>
</project>
```
  application.yml
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
  正常编写 Controller

##### 6.2 Zookeeper 服务消费者
  pom 引入依赖,与上述服务提供者一致  
  application.yml，与上述服务提供者一致  
  开启注解，注入 restTemplate，开启负载均衡  
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
  调用服务提供者  
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
  cocnsul 是 Go 语言编写的一款注册中心服务，<https://www.consul.io/intro/index.html>  
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
  pom 引入依赖
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
        </dependency>
    </dependencies>
</project>
```
  application.yml
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
  正常编写 Controller

##### 7.2 Consul 服务消费者
  pom 引入依赖,与上述服务提供者一致  
  application.yml，与上述服务提供者一致  
  开启注解，注入 restTemplate，开启负载均衡  
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
  调用服务提供者  
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
  Ribbon是Netflix公司开源的一个负载均衡的项目（<https://github.com/Netflix/ribbon>），它是一个基于HTTP、 TCP的客户端负载均衡器。

1. 什么是负载均衡？ 负载均衡是微服务架构中必须使用的技术，通过负载均衡来实现系统的高可用、集群扩容等功能。负载均衡可通过
硬件设备及软件来实现，硬件比如：F5、Array等，软件比如：LVS、Nginx等。  
用户请求先到达负载均衡器（也相当于一个服务），负载均衡器根据负载均衡算法将请求转发到微服务。
负载均衡算法有：轮训、随机、加权轮训、加权随机、地址哈希等方法，负载均衡器维护一份服务列表，
根据负载均衡算法 将请求转发到相应的微服务上，所以负载均衡可以为微服务集群分担请求，降低系统的压力。

2. 什么是客户端负载均衡？ 客户端负载均衡与服务端负载均衡的区别在于客户端要维护一份服务列表，
Ribbon从 Eureka Server获取服务列表，Ribbon根据负载均衡算法直接请求到具体的微服务，中间省去了负载均衡服务。

  Spring Cloud引入Ribbon配合 restTemplate 实现客户端负载均衡。Java中远程调用的技术有很多，
如： webservice、socket、rmi、Apache HttpClient、OkHttp等，互联网项目使用基于http的客户端较多。

  pom 引入依赖
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

  application.yml 配置 ribbon ,默认超时时间是 1 秒
```yml
ribbon:
  MaxAutoRetries: 2 #最大重试次数，当Eureka中可以找到服务，但是服务连不上时将会重试
  MaxAutoRetriesNextServer: 3 #切换实例的重试次数
  OkToRetryOnAllOperations: false #对所有操作请求都进行重试，如果是get则可以，如果是post，put等操作没有实现幂等的情况下是很危险的,所以设置为false
  ConnectTimeout: 5000 #请求连接的超时时间
  ReadTimeout: 6000 #请求处理的超时时间
```

  定义RestTemplate，使用@LoadBalanced注解
```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate(new OkHttp3ClientHttpRequestFactory());
}
```

  测试代码
```java
@Test
public void testRibbon() {
    String serviceId = "xxxxx";
    for (int i = 0; i < 10; i++) {
        ResponseEntity<Map> forEntity = restTemplate.getForEntity("http://"+serviceId+"/page/get/5a754adf6abb500ad05688d9", Map.class);
        Map body = forEntity.getBody();
    }
}
```

#### 9. OpenFeign 服务接口调用
  Feign是Netflix公司开源的轻量级rest客户端，使用Feign可以非常方便的实现Http 客户端。Spring Cloud引入 Feign并且集成了Ribbon实现客户端负载均衡调用。  

Feign工作原理如下：
1. 启动类添加@EnableFeignClients注解，Spring会扫描标记了@FeignClient注解的接口，并生成此接口的代理对象
2. @FeignClient(value = "xxx")即指定了cms的服务名称，Feign会从注册中 心获取cms服务列表，并通过负载均衡算法进行服务调用。
3. 在接口方法 中使用注解@GetMapping("/page/get/{id}")，指定调用的url，Feign将根据url进行远程调用。 

Feign注意点
SpringCloud对Feign进行了增强兼容了SpringMVC的注解 ，我们在使用SpringMVC的注解时需要注意：
1. feignClient接口 有参数在参数必须加@PathVariable("XXX")和@RequestParam("XXX")
2. feignClient返回值为复杂对象时其类型必须有`无参构造函数`。

  openFeign 中内置了 ribbon ，负载均衡默认采用轮询算法，默认超时参数为 1 秒
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
##### 9.1 OpenFeign 配置
  pom 引入依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

  application.yml 配置参数
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
    enabled: true

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
```yaml
# ribbon参数也可以直接配在feignClient下面，相较于ribbon：可能会美观一点
feign:
  hystrix:
    enabled: true
  client:
    config:
      default:
        connectTimeout: 50000
        readTimeout: 50000
        maxAutoRetries: 2
        maxAutoRetriesNextServer: 3
        okToRetryOnAllOperations: false
        loggerLevel: full
```

  启动类上标识开启 openFeign
```java
@EnableDiscoveryClient
@EnableFeignClients
@EnableHystrix
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class MyApplication {
}
```

##### 9.2 OpenFeign 配置日志
```java
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        //这里记录所有，根据实际情况选择合适的日志level
        return Logger.Level.FULL;
    }
}
```

##### 9.3 OpenFeign 服务降级
  统一配置 feignClient, value=微服务名字, contextId 自定义需要唯一，建议微服务名+Client名, fallback=降级时调用对应类方法  
`SysDepartmentClientFallBack.class` 需实现该 client 接口的所有方法并且加上 `@component` 注解, configuration 可以指定配置
```java
@FeignClient(value = "PONY-API-ORIGINAL-STATIC", contextId = "PONY-API-ORIGINAL-STATIC-SysDepartmentClient", fallback = SysDepartmentClientFallBack.class, configuration = FeignConfig.class)
public interface SysDepartmentClient {

    @PostMapping("/department/insert")
    public boolean insert(@RequestBody SysDepartment department);

    @PostMapping("/department/insertBatch")
    public boolean insertBatch(@RequestBody List<SysDepartment> departmentList);

    @PutMapping("/department/updateById")
    public boolean updateById(@RequestBody SysDepartment department);

    @DeleteMapping("/department/delete/{id}")
    public boolean deleteById(@PathVariable("id") String id);

    @GetMapping("/department/get/{id}")
    public SysDepartment getById(@PathVariable("id") String id);

    @GetMapping("/department/getAll")
    public List<SysDepartment> getAll();

    @PostMapping("/department/search")
    public List<SysDepartment> search(@RequestBody Map<String, Object> params);

    @PostMapping("/department/searchPage/{page}/{size}")
    public Page<SysDepartment> searchPage(@RequestBody Map<String, Object> params, @PathVariable("page") int page, @PathVariable("size") int size);

}
```



#### 10. Hystrix 断路器
##### Hystrix介绍
  在微服务场景中，通常会有很多层的服务调用。如果一个底层服务出现问题，故障会被向上传播给用户。我们需要一种机制，当底层服务不可用时，可以阻断故障的传播。这就是断路器的作用。他是系统服务稳定性的最后一重保障。
  在springcloud中断路器组件就是Hystrix。Hystrix也是Netflix套件的一部分。他的功能是，当对某个服务的调用在一定的时间内（默认10s），有超过一定次数（默认20次）并且失败率超过一定值（默认50%），该服务的断路器会打开。返回一个由开发者设定的fallback。
  fallback可以是另一个由Hystrix保护的服务调用，也可以是固定的值。fallback也可以设计成链式调用，先执行某些逻辑，再返回fallback。  

##### 服务雪崩 (雪崩的时候没有一片❄是无辜的)
  多个微服务之间调用的时候，假如 微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的 扇出。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，从而引起系统崩溃，这就是所谓的 "雪崩效应"。
  对于高流量的应用来说，单一的后端依赖可能会导致 所有服务器 上的 所有资源 在几秒内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的 级联故障。这些都需要对故障和延迟就行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统的宕机。
  通常，当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然而这个问题模块还调用了其他模块，这就会导致级联故障，或者叫 雪崩。
  这种级联故障的避免，就需要有一种兜底的方案，或者一种链路终断的方案。这就是"服务降级"。

##### Hystrix作用

  Hystrix ，又称豪猪哥。是一个用于处理分布式系统的 延迟 和 容错 的开源库。在分布式系统中，许多依赖不可避免的会调用失败，比如：超时、异常等原因。Hystrix 能够保证在一个依赖出现问题的情况下， 不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。
  断路器 本身是一种开关装置。当某个服务单元发生故障之后，通过 断路器 的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用调用无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要的占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。
  Hystrix 的功能就是 服务降级、服务熔断、接近实时的监控、服务限流、服务隔离，最重要的还是前面三个功能。 Hystrix 官网使用介绍：https://github.com/Netflix/Hystrix/wiki/How-To-Use
  Hystrix 在 消费端、服务端 根据定义的规则，都能使用 服务降级、限流（比如，服务端约定等待3s 返回，但是客户端只让等待 2s，就可以再客户端 添加 Hystrix。这些都可以根据自己情况），一般用在消费端。
  默认配置：com.netflix.hystrix.HystrixCommandProperties 超时 1 秒会 fallback
```java
public abstract class HystrixCommandProperties {
    /* defaults */
    /* package */ static final Integer default_metricsRollingStatisticalWindow = 10000;// default => statisticalWindow: 10000 = 10 seconds (and default of 10 buckets so each bucket is 1 second)
    private static final Integer default_metricsRollingStatisticalWindowBuckets = 10;// default => statisticalWindowBuckets: 10 = 10 buckets in a 10 second window so each bucket is 1 second
    private static final Integer default_circuitBreakerRequestVolumeThreshold = 20;// default => statisticalWindowVolumeThreshold: 20 requests in 10 seconds must occur before statistics matter
    private static final Integer default_circuitBreakerSleepWindowInMilliseconds = 5000;// default => sleepWindow: 5000 = 5 seconds that we will sleep before trying again after tripping the circuit
    private static final Integer default_circuitBreakerErrorThresholdPercentage = 50;// default => errorThresholdPercentage = 50 = if 50%+ of requests in 10 seconds are failures or latent then we will trip the circuit
    private static final Boolean default_circuitBreakerForceOpen = false;// default => forceCircuitOpen = false (we want to allow traffic)
    /* package */ static final Boolean default_circuitBreakerForceClosed = false;// default => ignoreErrors = false 
    private static final Integer default_executionTimeoutInMilliseconds = 1000; // default => executionTimeoutInMilliseconds: 1000 = 1 second
    private static final Boolean default_executionTimeoutEnabled = true;
    private static final ExecutionIsolationStrategy default_executionIsolationStrategy = ExecutionIsolationStrategy.THREAD;
    private static final Boolean default_executionIsolationThreadInterruptOnTimeout = true;
    private static final Boolean default_executionIsolationThreadInterruptOnFutureCancel = false;
    private static final Boolean default_metricsRollingPercentileEnabled = true;
    private static final Boolean default_requestCacheEnabled = true;
    private static final Integer default_fallbackIsolationSemaphoreMaxConcurrentRequests = 10;
    private static final Boolean default_fallbackEnabled = true;
    private static final Integer default_executionIsolationSemaphoreMaxConcurrentRequests = 10;
    private static final Boolean default_requestLogEnabled = true;
    private static final Boolean default_circuitBreakerEnabled = true;
    private static final Integer default_metricsRollingPercentileWindow = 60000; // default to 1 minute for RollingPercentile 
    private static final Integer default_metricsRollingPercentileWindowBuckets = 6; // default to 6 buckets (10 seconds each in 60 second window)
    private static final Integer default_metricsRollingPercentileBucketSize = 100; // default to 100 values max per bucket
    private static final Integer default_metricsHealthSnapshotIntervalInMilliseconds = 500; // default to 500ms as max frequency between allowing snapshots of health (error percentage etc)
}
```


##### 10.1 服务降级
  当服务器忙时，友好提示客户 "请稍候再试" ，不让客户端处于一直等待状态，并立刻返回一个友好提示。
当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作。
  比如电商平台，在针对 618、双11 等高峰情形下采用的部分服务不出现或者延时出现的情形。比较典型的就是支付，在0点进行支付，由于大量请求的集中涌入，服务器压力瞬间过大，从而导致数据返回超时等情况，这时就需要对支付模块进行服务降级处理。比如说：接口超过3s没有返回数据，就提示"数据加载失败，被挤爆了的提示"，在中断当前支付请求的同时，进行了友好的提示。

##### 10.2 服务熔断
  服务熔断：类似于我们家用的保险丝，当某服务出现不可用或响应超时的情况时，为了防止整个系统出现雪崩，暂时 停止对该服务的调用 。过一段时间，服务器会慢慢进行恢复，直到完全恢复服务提供。
服务降级和服务熔断的区别：
+ 服务降级 → 当前服务还是可用的；（比如有10个线程，谁抢到谁用，抢不到如果超时报提错误提示，下一次抢到还能继续提供服务）
+ 服务熔断 → 当前服务不可用，但是它会逐渐恢复服务提供；(拉闸，整个家用电器都不能用；然后测试开5个电器没问题，6个也没问题，逐渐的就恢复服务提供)

##### 10.3 服务限流
  服务限流场景：一般应用在 秒杀，高并发 等操作。严禁一窝蜂的过来拥挤，大家排队，一秒钟只允许通过 N 个，有序进行。

##### 简单配置使用
  引入相关依赖并开启注解 `@EnableCircuitBreaker`
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
  在需要降级/熔断/限流的方法上加注解，优先级： method > class > config
```java
@HystrixCommand(
        fallbackMethod = "method_fallback",
        commandProperties = {
                @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),// 是否开启断路器
                @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "3"),// 请求次数
                @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),// 时间窗口期/时间范文
                @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50")// 失败率达到多少后跳闸
        },
        threadPoolProperties = {
                // @HystrixProperty(name = "xx", value = "xx")
        }
)
public String method(@PathVariable("id") Integer id) {
    return "ok";
}

public String method_fallback(@PathVariable("id") Integer id) {
    return "fallback";
}
```
  每个方法都有一个 fallback_method , 未免有些代码膨胀, 况且业务方法 和 自定义降级方法 混合在一起(业务逻辑方法 和 处理服务降级、熔断方法 揉在一块)
  解决方法：使用 @DefaultProperties(defaultFallback = "xxx") 的方式，定义全局 服务降级 方法, @HystrixCommand 注解加在对应方法上即可
```java
@DefaultProperties(defaultFallback = "global_fallback_method") // 使用@DefaultProperties 定义全局服务降级方法
public class HystrixController {
    // 定义全局服务降级方法(不能有参数)
    public String global_fallback_method(){
        return "~~~~我是,全局服务降级方法";
    }
}

```



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

##### 10.5 Hystrix 统一配置
```java
@Controller
public class HelloController {

  @GetMapping("hello/{name}")
  @ResponseBody
  public Object hello(@PathVariable("name") String name) {
    System.out.println("hello request come in. timestamp:" + System.currentTimeMillis());
    return service.doSth(name);
  }

  @Resource
  private HelloService service;
}

@Service
public class HelloService {

  @HystrixCommand(groupKey = "helloGroup", commandKey = "hello")
  public String doSth(String username) {
    try {
      Thread.sleep(2000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    return username + " do something.";
  }
}
```
  最简单的方式就是通过HystrixCommand注解来启用Hystrix，但是这里比较麻烦的是，多个方法都要使用HystrixCommand，那注解每个都要写，
而且同一个组内的command应该公用同一个配置，那么请看下面的application.yml的配置，解决问题
调用第三方系统helloGroup走HyStrixCommand
```yaml
hystrix:
  threadpool:
    helloGroup:
      coreSize: 10
      maximumSize: 10
      maxQueueSize: -1
```
  针对不同的组在配置文件里面加上不同的配置就好了，在@MyCommand注解里面指定group为abc就行；其他的配置也是这个规则，还有默认的配置是default；
这样可以把一个组的配置独立出来，便于配置，而且开发者也会方便很多，代码简洁；下面附上所有的配置项供参考

```yaml
#default可替换
hystrix:
  command:
    default:
      execution:
        isolation:
          #线程池隔离还是信号量隔离 默认是THREAD 信号量是SEMAPHORE
          strategy: THREAD
          semaphore:
            #使用信号量隔离时，支持的最大并发数 默认10
            maxConcurrentRequests: 10
          thread:
            #command的执行的超时时间 默认是1000
            timeoutInMilliseconds: 2000
            #HystrixCommand.run()执行超时时是否被打断 默认true
            interruptOnTimeout: true
            #HystrixCommand.run()被取消时是否被打断 默认false
            interruptOnCancel: false
        timeout:
          #command执行时间超时是否抛异常 默认是true
          enabled: true
        fallback:
          #当执行失败或者请求被拒绝，是否会尝试调用hystrixCommand.getFallback()
          enabled: true
          isolation:
            semaphore:
              #如果并发数达到该设置值，请求会被拒绝和抛出异常并且fallback不会被调用 默认10
              maxConcurrentRequests: 10
      circuitBreaker:
        #用来跟踪熔断器的健康性，如果未达标则让request短路 默认true
        enabled: true
        #一个rolling window内最小的请求数。如果设为20，那么当一个rolling window的时间内
        #（比如说1个rolling window是10秒）收到19个请求，即使19个请求都失败，也不会触发circuit break。默认20
        requestVolumeThreshold: 5
        # 触发短路的时间值，当该值设为5000时，则当触发circuit break后的5000毫秒内
        #都会拒绝request，也就是5000毫秒后才会关闭circuit，放部分请求过去。默认5000
        sleepWindowInMilliseconds: 5000
        #错误比率阀值，如果错误率>=该值，circuit会被打开，并短路所有请求触发fallback。默认50
        errorThresholdPercentage: 50
        #强制打开熔断器，如果打开这个开关，那么拒绝所有request，默认false
        forceOpen: false
        #强制关闭熔断器 如果这个开关打开，circuit将一直关闭且忽略
        forceClosed: false
      metrics:
        rollingStats:
          #设置统计的时间窗口值的，毫秒值，circuit break 的打开会根据1个rolling window的统计来计算。若rolling window被设为10000毫秒，
          #则rolling window会被分成n个buckets，每个bucket包含success，failure，timeout，rejection的次数的统计信息。默认10000
          timeInMilliseconds: 10000
          #设置一个rolling window被划分的数量，若numBuckets＝10，rolling window＝10000，
          #那么一个bucket的时间即1秒。必须符合rolling window % numberBuckets == 0。默认10
          numBuckets: 10
        rollingPercentile:
          #执行时是否enable指标的计算和跟踪，默认true
          enabled: true
          #设置rolling percentile window的时间，默认60000
          timeInMilliseconds: 60000
          #设置rolling percentile window的numberBuckets。逻辑同上。默认6
          numBuckets: 6
          #如果bucket size＝100，window＝10s，若这10s里有500次执行，
          #只有最后100次执行会被统计到bucket里去。增加该值会增加内存开销以及排序的开销。默认100
          bucketSize: 100
        healthSnapshot:
          #记录health 快照（用来统计成功和错误绿）的间隔，默认500ms
          intervalInMilliseconds: 500
      requestCache:
        #默认true，需要重载getCacheKey()，返回null时不缓存
        enabled: true
      requestLog:
        #记录日志到HystrixRequestLog，默认true
        enabled: true
  collapser:
    default:
      #单次批处理的最大请求数，达到该数量触发批处理，默认Integer.MAX_VALUE
      maxRequestsInBatch: 2147483647
      #触发批处理的延迟，也可以为创建批处理的时间＋该值，默认10
      timerDelayInMilliseconds: 10
      requestCache:
        #是否对HystrixCollapser.execute() and HystrixCollapser.queue()的cache，默认true
        enabled: true
  threadpool:
    default:
      #并发执行的最大线程数，默认10
      coreSize: 10
      #Since 1.5.9 能正常运行command的最大支付并发数
      maximumSize: 10
      #BlockingQueue的最大队列数，当设为－1，会使用SynchronousQueue，值为正时使用LinkedBlcokingQueue。
      #该设置只会在初始化时有效，之后不能修改threadpool的queue size，除非reinitialising thread executor。
      #默认－1。
      maxQueueSize: -1
      #即使maxQueueSize没有达到，达到queueSizeRejectionThreshold该值后，请求也会被拒绝。
      #因为maxQueueSize不能被动态修改，这个参数将允许我们动态设置该值。if maxQueueSize == -1，该字段将不起作用
      queueSizeRejectionThreshold: 5
      #Since 1.5.9 该属性使maximumSize生效，值须大于等于coreSize，当设置coreSize小于maximumSize
      allowMaximumSizeToDivergeFromCoreSize: false
      #如果corePoolSize和maxPoolSize设成一样（默认实现）该设置无效。
      #如果通过plugin（https://github.com/Netflix/Hystrix/wiki/Plugins）使用自定义实现，该设置才有用，默认1.
      keepAliveTimeMinutes: 1
      metrics:
        rollingStats:
          #线程池统计指标的时间，默认10000
          timeInMilliseconds: 10000
          #将rolling window划分为n个buckets，默认10
          numBuckets: 10
```

##### 10.6 Hystrix Dashboard
  Hystrix Dashboard 是豪猪哥的监控面板，可以监控服务状态，被监控的服务需有 actuator 依赖引入

  pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

  启动类加注解
```java
@EnableHystrixDashboard
@SpringBootApplication
public class Dashboard9001Application {
    public static void main(String[] args) {
        SpringApplication.run(Dashboard9001Application.class, args);
    }
}
```
  此时可以访问 dashboard 了 http://localhost:9001/hystrix  
![dashboard1](https://gitee.com/clownfish7/image/raw/master/hystrix/dashboard1.png 'dashboard1')
  如果连不上对应服务，需要在需要监控的服务上开启一个指标输出，如果连上了一直在 loading 图表则说明该服务没指标数据，需要访问几次服务接口即可
```java
/**
 * 此配置是为了服务监控而配置，与服务容错本身无观，springCloud 升级之后的坑
 * ServletRegistrationBean因为springboot的默认路径不是/hystrix.stream
 * 只要在自己的项目中配置上下面的servlet即可
 *
 * @return
 */
@Bean
public ServletRegistrationBean getServlet() {
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean<HystrixMetricsStreamServlet> registrationBean = new ServletRegistrationBean<>(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```
![dashboard2](https://gitee.com/clownfish7/image/raw/master/hystrix/dashboard2.png 'dashboard2')
![dashboard3](https://gitee.com/clownfish7/image/raw/master/hystrix/dashboard3.png 'dashboard3')


#### 11. Zuul 网关
  Zuul包含了对请求的路由和过滤两个最主要的功能:
> 其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础.
  而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础.
  Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。  
> 官网资料：<https://github.com/Netflix/zuul/wiki/Getting-Started>  

  pom 引入依赖
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

  yml 配置
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

  开启注解
```java
@SpringBootApplication
@EnableZuulProxy//此工程是一个zuul网关
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

  zuul 过滤器
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
> Gateway官网: <https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/>

  Cloudquan全家桶中有个很重要的组件就是网关，在1.x版本中采用的都是 Zuul 网关；但在2.x版本中，
zuul 的升级一直跳票，SpringCloud 最后自己研发了一个网关替代 zuul；那就是 SpringCloud-Gateway。

  SpringCloud Gateway 是 SpringCloud 的一个全新项目，基于 Spring5.0 + SpringBoot2.0 和 Project Reactor 等技术开发的网关，
它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。
  SpringCloud Gateway 作为 SpringCloud 生态系统中的网关，目标是替代 Zuul，在 SpringCloud2.0 以上版本中，没有对 Zuul2.0 以上最新
高性能版本进行集成，仍然还是使用的 Zuul1.x 非 Reactor 模式的老版本。而为了提升网关的性能，SSpringCloud Gateway 是基于 WebFlux 框架实现的，
而 WebFlux 框架底层则使用了高性能的 Reactor 模式的通信框架 Netty。
  SpringCloud Gateway 的目标提供统一的路由方式且基于 Filter 链的方式提供了网关的基本功能，例如安全、监控/指标和限流。

##### 12.1 Gateway 具有的特性
+ 基于 Spring Framework 5、Project Reactor 和 Spring Boot 2.0 进行构建；
+ 动态路由：能够匹配任何请求属性；
+ 可以对路由指定 Predicate(断言) 和 Filter(过滤器)，易于编写；
+ 集成 Hystrix 的断路器功能；
+ 集成 Spring Cloud 服务发现功能（Gateway一样可注册到 Eureka）；
+ 请求限流功能；
+ 支持路径重写；
+ Spring 自家产品，更稳定。
##### 12.2 Gateway 和 Zuul 的区别
  在 Spring Cloud Finchley 正式版之前，Spring Cloud 推荐的网关是 Netflix 提供的 Zuul。在这之后，还是更倾向于推荐使用自己的亲儿子：Spring Cloud Gateway。 它两者区别如下：
1. Zuul 1.x 是一个基于 阻塞 I/O 的 API 网关
2. Zuul 1.x 基于 Servlet 2.5 使用阻塞架构，它不支持任何长连接（如：websocket）。Zuul的设计模式和 Nginx 类似，每次 I/O 操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是 Nginx 用 C++ 实现，Zuul 用 Java 实现，JVM 本身会有第一次加载较慢的情况，是的 Zuul 的性能相对较差；
3. Zuul 2.x 理念更先进，但是 Spring Cloud 目前还没有整合。Zuul 2.x 的性能较 Zuul 1.x 有较大提升。在性能方面，根据官方提供的基准测试，Spring Cloud Gateway 的 RPS(每秒请求数)是 Zuul 的 1.6 倍；
4. Spring Cloud Gateway 建立在 Spring Framework 5、Project Reactor 和 Spring Boot 2.0 之上，使用非阻塞 API；
5. Spring Cloud Gateway 还支持 WebSocket，并且与 Spring 集成会有更好的开发体验。
##### 12.3 Gateway 三大概念
  Route(路由)、Predicate(断言)、Filter（过滤），这三大概念构成了强大的 Gateway。一个web 请求，通过一些匹配条件，最终定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。Predicate 就是我们的匹配条件；而 Filter 就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标 URI，就可以实现一个具体的路由了。
+ Route(路由)
  + 路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为 true 则匹配该路由
+ Predicate(断言)
  + 参考的是 java8 的 java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求URL与断言相匹配（true）则进行路由 
+ Filter(过滤)
  + 指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。类似 过滤器、拦截器 的概念。
##### 12.4 Gateway 工作流程
![gateway1](https://gitee.com/clownfish7/image/raw/master/gateway/gateway1.png 'gateway1')

![gateway2](https://gitee.com/clownfish7/image/raw/master/gateway/gateway2.png 'gateway2')
##### 12.5 Gateway 使用方式
  pom.xml 无需引入 web ！
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

  启动类注解
```java
@EnableDiscoveryClient
@SpringBootApplication
public class Gateway9527Application {
    public static void main(String[] args) {
        SpringApplication.run(Gateway9527Application.class, args);
    }
}
```

  application.yml
```yml
server:
  port: 9527

# 单机版
spring:
  application:
    name: cloud-gateway
  cloud:
    # gateway 网关配置
    gateway:
      routes:
        #路由的ID，没有固定规则但要求唯一，建议配合服务名
        - id: payment_route
          #匹配后提供服务的路由地址
          uri: http://localhost:8001
          #断言
          predicates:
              # 路径相匹配的进行路由
            - Path=/payment/get/**
        #路由的ID，没有固定规则但要求唯一，建议配合服务名
        - id: payment_route_lb
          #匹配后提供服务的路由地址
          uri: http://localhost:8001
          predicates:
            # 路径相匹配的进行路由
            - Path=/payment/get/lb
# 集群版
spring:
  application:
    name: cloud-gateway
  cloud:
    # gateway 网关配置
    gateway:
      discovery:
        locator:
          #开启从注册中心动态创建路由的功能，利用微服务名进行路由
          enabled: true
      routes:
        #路由的ID，没有固定规则但要求唯一，建议配合服务名
        - id: payment_route
          #匹配后提供服务的路由地址 #集群服务路由地址
          uri: lb://cloud-payment-service
          #断言
          predicates:
            # 路径相匹配的进行路由
            - Path=/payment/get/**
        #路由的ID，没有固定规则但要求唯一，建议配合服务名
        - id: payment_route_lb
          #匹配后提供服务的路由地址 #集群服务路由地址
          uri: lb://cloud-payment-service
          predicates:
            # 路径相匹配的进行路由
            - Path=/payment/get/lb
            # 时间在此之后的进行路由 日期可以使用 ZonedDateTime.now();
            - After=2020-08-29T16:47:09.015+08:00[Asia/Shanghai]
            # 时间在此之前的进行路由 日期可以使用 ZonedDateTime.now();
            - Before=2020-08-29T16:47:09.015+08:00[Asia/Shanghai]
            # 时间在此之间的进行路由 日期可以使用 ZonedDateTime.now();
            - Between=2020-08-29T16:47:09.015+08:00[Asia/Shanghai],2020-08-29T16:47:09.015+08:00[Asia/Shanghai]
            # cookie 包含 key=key，value 匹配正则表达式 aa 的进行路由
            - Cookie=key, aa
            # header 包含 key=X-Request-Id，value 匹配正则表达式 \d+ 的进行路由
            - Header=X-Request-Id, \d+
            # host 匹配的进行路由
            - Host=**.somehost.org,**.anotherhost.org
            # 请求方法为 Get/Post 的进行路由
            - Method=GET,POST
            # remoteAddr 匹配的进行路由
            - RemoteAddr=192.168.1.1/24

eureka:
  instance:
    hostname: localhost
    #所在主机ip
    ip-address: 127.0.0.1
    #将自己的ip地址注册到Eureka服务中
    prefer-ip-address: true
    #指定实例id
    instance-id: ${spring.application.name}:${server.port}
    #eureka客户端向服务端发送心跳时间间隔，默认30s
    lease-renewal-interval-in-seconds: 30
    #eureka服务端在收到最后一次心跳后的等待时间，超时将删除服务，90s
    lease-expiration-duration-in-seconds: 90
  client:
    # 表示是否注册进eureka,默认为true
    register-with-eureka: true
    # 表示是否从EurekaServer抓取已有注册信息，默认为true，单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
```

###### 12.5.1 Gateway 代码配置断言方式
  Gateway 网关模块启动时，在 Console 工作台能够观察到 Gateway 去加载 Predicate。我们会看到 Gateway 一共提供了 11 种 Predicate 机制(xxxFactory、xxxService 2个除外)。
```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("toBaiduGuoNei", r -> r.path("/guonei").uri("https://news.baidu.com/guonei"))
                .build();
    }
}
```
###### 12.5.2 Gateway 代码配置自定义过滤器方式
  自定义全局过滤器，可以帮助我们进行 全局日志记录、统一网关鉴权 等功能。我们需要定义一个类，并实现 GlobalFilter ，Ordered 两个接口，重写里面的 filter、order 方法即可。
  主要是 filter 方法，编写详细的过滤器逻辑；order 方法是用来定义加载过滤器的优先级，返回一个 int 数值，值越小优先级越高。

```java
@Slf4j
@Component
public class GatewayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("*********come in MyLogGateWayFilter: "+new Date());
        log.info("*********进入全局过滤器： "+new Date());
        String username = exchange.getRequest().getQueryParams().getFirst("username");
        if (StringUtils.isEmpty(username)) {
            log.info("*****用户名为Null 非法用户,(┬＿┬)");
            //给人家一个回应，设置http状态码
            exchange.getResponse()
                    .setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        //加载过滤器优先级，越小优先级越高
        return 0;
    }
}
```

#### 13. SpringCloud config 分布式配置中心
  Spring Cloud Config 为[wèi]微服务架构中的微服务 **`提供集中化的外部配置支持`**，配置服务器为[wèi]各个不同微服务应用的所有环境 **`提供了一个中心化的外部配置`**。公共配置都去配置中心读取，私有配置，各个服务独自配置，简直不要太爽
##### 13.1 SpringCloud config 是什么
  Spring Cloud Config 分为 服务端 和 客户端。
+ 服务端：也称为 分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。就是图中的 Config Server；
+ 客户端：通过指定的 配置中心(Config Server) 来管理应用资源，以及与业务相关的配置内容，并在启动的时候从 配置中心 获取和加载配置信息。

  服务器默认采用 Git 来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过 Git 客户端工具来方便的管理和访问配置内容。(服务器也支持其他方式：支持SVN 和 本地文件，最推荐 Git，而且使用的是 http/https 访问的形式)
##### 13.2 SpringCloud config 结构图
![config1](https://gitee.com/clownfish7/image/raw/master/springcloud-config/config1.png 'config1')
![config2](https://gitee.com/clownfish7/image/raw/master/springcloud-config/config2.png 'config2')
##### 13.3 SpringCloud config 服务端配置
**`在远程仓库(github/gitee/svn)先创建好对应目录及配置文件`**
```yml
#/springcloud-config/config-dev.yml
config:
  info: "master branch, springcloud-config/config-dev.yml, version=1"
#/springcloud-config/config-prod.yml
config:
  info: "master branch, springcloud-config/config-prod.yml, version=1"
#/springcloud-config/config-test.yml
config:
  info: "master branch, springcloud-config/config-test.yml, version=1"
```
  pom.xml
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
  启动类注解
```java
@EnableConfigServer
@SpringBootApplication
public class ConfigCenter3344Application {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenter3344Application.class, args);
    }
}
```
  application.yml
```yml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          #GitHub远程仓库地址
          uri: https://gitee.com/clownfish7/springcloud2020.git
          # 搜索目录
          search-paths: springcloud-config
      #读取分支
      label: master
eureka:
  instance:
    hostname: localhost
    #所在主机ip
    ip-address: 127.0.0.1
    #将自己的ip地址注册到Eureka服务中
    prefer-ip-address: true
    #指定实例id
    instance-id: ${spring.application.name}:${server.port}
    #eureka客户端向服务端发送心跳时间间隔，默认30s
    lease-renewal-interval-in-seconds: 30
    #eureka服务端在收到最后一次心跳后的等待时间，超时将删除服务，90s
    lease-expiration-duration-in-seconds: 90
  client:
    # 表示是否注册进eureka,默认为true
    register-with-eureka: true
    # 表示是否从EurekaServer抓取已有注册信息，默认为true，单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
```
  启动 Config 模块后，测试通过 Config 微服务是否可以从 远程仓库上获取配置内容。我们通过地址：http://localhost:3344/config-dev.yml 进行配置内容的访问。
```yml
# response
config:
  info: master branch, springcloud-config/config-dev.yml, version=1
```
##### 13.4 GitHub/Gitee 配置文件读取规则
   远程 GitHub 仓库，配置文件的命名也是有具体规则的。Spring Cloud Config 官方共支持 5 种方式的配置。5种配置规则见：[Config 官网配置规则](https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/#_quick_start, 'Config 官网配置规则')
+ /{application}/{profile}/{label}
+ /{application}-{profile}.yml (这种不带label方式，默认使用 application.yml 配置)
+ /{label}/{application}-{profile}.yml (推荐使用第三种)
+ /{application}-{profile}.properties
+ /{label}/{application}-{profile}.properties
  规则说明:
1. /{application}/{profile}/{label} 这种方式，返回的是 Json 对象，需要自己解析所要的内容；
2. /{application}-{profile}.yml 这种不带 label 方式，因为 applicaiton.yml 文件已经有配置过 label，不带label 方式，默认走的就是 yml 配置的 label，返回的是配置内容；
3. /{label}/{application}-{profile}.yml **`推荐使用第3种`**，这种方式简明扼要，条理清晰，返回的是配置内容；
4. {application}-{profile}.properties 同第2种；
5. {label}/{application}-{profile}.properties 同第3种。
  参数说明:
1. label：GitHub 分支(branch)名称
2. application：服务名
3. profile：环境(dev/test/prod)
##### 13.4 SpringCloud config 客户端配置
  客服端：在启动的时候从 配置中心(Config Server) 获取和加载配置信息。

  pom.xml
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
  启动类注解
```java
@EnableDiscoveryClient
@SpringBootApplication
public class ConfigClient3355Application {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient3355Application.class, args);
    }
}
```
  配置文件 **`bootstrap.yml`**  
  application.yml 是(用户级)的资源配置项。bootstrap.yml 是(系统级)的资源配置项，优先级更高。

  Spring Cloud 会创建一个 “Bootstrap Context”，作为 Spring 应用的 “Application Context” 的 父上下文。初始化的时候，“Bootstrap Context” 负责从 外部源 加载配置属性并解析配置。这两个上下文共享一个从外部获取的 “Environment”。

  "Bootstrap" 属性有高优先级。默认情况下，它们不会被本地配置覆盖。 “Bootstrap Context” 和 “Application Context” 有这不同的约定，所以新增了一个 "bootstrap.yml" 文件，保证"Bootstrap Context" 和 "Application Context" 配置的分离。

  所以，将客户端模块下的 application.yml 文件改为bootstrap.yml，这是很关键的。 因为 bootstrap.yml 是比 application.yml 优先加载的。bootstrap.yml 优先级高于 applicaiton.yml。
```yml
server:
  port: 3355
spring:
  application:
    name: cloud-config-client
  cloud:
    #config客户端配置
    config:
      #分支名称
      label: master
      #配置文件名称
      name: config
      # 读取后缀名称 3个综合：master分支上config-dev.yml 的配置文件被读取(http://config-3344.com:3344/master/config-dev.yml)
      profile: dev
      uri: http://cloud-config-center   #配置中心地址 http://host:port
      discovery:
        # 开启从注册中心发现 config-server
        enabled: true
        # config-server name
        service-id: cloud-config-center
eureka:
  instance:
    hostname: localhost
    #所在主机ip
    ip-address: 127.0.0.1
    #将自己的ip地址注册到Eureka服务中
    prefer-ip-address: true
    #指定实例id
    instance-id: ${spring.application.name}:${server.port}
    #eureka客户端向服务端发送心跳时间间隔，默认30s
    lease-renewal-interval-in-seconds: 30
    #eureka服务端在收到最后一次心跳后的等待时间，超时将删除服务，90s
    lease-expiration-duration-in-seconds: 90
  client:
    # 表示是否注册进eureka,默认为true
    register-with-eureka: true
    # 表示是否从EurekaServer抓取已有注册信息，默认为true，单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
```
  配置一个 Controller 便于查看结果
```java
@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;
    @GetMapping("/config/info")
    public String configInfo() {
        return configInfo;
    }
}
```
  启动 Config-Client 模块后，测试通过 Config 微服务是否可以从 远程仓库上获取配置内容。我们通过地址：http://localhost:3355/config/info 进行配置内容的访问。  
  response: master branch, springcloud-config/config-dev.yml, version=1
##### 13.5 配置的动态刷新问题
  当 GitHub 上的配置文件内容有调整，Github中配置变更后，ConfigServer 配置中心会立刻响应，然鹅客户端却没有任何响应，除非客户端重启或者重新加载，才能够获取最新的配置。 难道每次修改配置文件，客户端都需要重启吗？？

  那简直就是个噩梦，还是没有解决根本问题。为了避免每次修改 GitHub 配置文件后，客户端都需要重启的问题，此处就引出了客户端 动态刷新 的问题。

接下来对客户端进行 动态刷新 配置。

  pom.xml 引入actuator监控
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
  bootstrap.yml，新增暴露监控端口配置
```yml
#暴露监控端点
management:
  endpoints:
    web:
      exposure:
        # 此处有很多选项可以配置，直接配置 * 代表包含全部
        include: "*"
```
  Controller层添加 @RefreshScope 注解  
  @RefreshScope  注解能帮助我们做局部的参数刷新，但侵入性较强，需要开发阶段提前预知可能的刷新点，
并且该注解底层是依赖于cglib进行代理的，所以不要掉入cglib的坑，出现刷了也不更新情况；
```java
@RefreshScope
@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;
    @GetMapping("/config/info")
    public String configInfo() {
        return configInfo;
    }
}
```
  修改完配置文件后，curl -X POST http://localhost:3355/actuator/refresh 刷新配置（**POST**）
##### 13.6 手动版动态刷新，存在的问题
  实现了动态刷新，解决了 ConfigClient 重启才能获取最新配置信息问题。假如有 N 多个台，就需要 N 多次的 curl -X POST "http://微服务地址:端口号/actuator/refresh"。这仍然是一个噩梦，还是没有解决根本问题。

  大规模 微服务/集群模式，我们可以采用广播的方式，一次通知，处处生效。类似于 消息队列的 Topic ，微信公众号 的概念，一次订阅，所有订阅者都能接收到新消息。

  他来了，他来了，Spring Cloud Bus 总线 带着 消息队列/广播 机制 向我们走来了。Spring Cloud Bus 总线 可以帮我们实现以下功能：

1. 真正实现：一处通知、处处生效；
2. 实现精确通知,只通知集群中的某些服务(精确通知，比如有100台机器，只通知前98台)

#### 14. SpringCloud Bus 消息总线
  官网：[Spring Cloud Bus 官网](https://cloud.spring.io/spring-cloud-static/spring-cloud-bus/2.2.1.RELEASE/reference/html/, 'Spring Cloud Bus 官网')

  在微服务架构的系统中，通常会使用 轻量级的消息代理 来构建一个共用的消息主题，并让系统中所有的微服务示例都连接上来。由于 该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便的广播一些需要让其他连接在该主题上的实例都知道的消息。

  Spring Cloud Bus 是用来将 分布式系统的节点与 轻量级消息系统连接起来的框架，它整合了 Java 的事件处理机制和消息中间件的功能。Spring Cloud Bus 目前仅支持 `RabbitMQ` 和 `Kafka`。
##### 14.1 SpringCloud Bus 原理
  Config 客户端示例，都去监听 MQ 中的同一个 topic（默认是 springCloudBus）。当一个服务刷新数据的时候，它会把这个消息放入到 Topic 中，这样其他监听同一 Topic 的服务就能够得到通知，然后去更新自身的配置。就是通过 MQ 消息队列的 Topic 机制，达到广播的效果。
![bus1](https://gitee.com/clownfish7/image/raw/master/springcloud-bus/bus1.png 'bus1')
##### 14.2 SpringCloud Bus 的两种设计思想
  选用 Spring Cloud Bus 进行 Topic 消息的发送，在技术选型上共有两种设计思想：
###### 触发客户端
  利用消息总线，触发一个客户端 的 /bus/refresh 端点。通过客户端向 Bus 总线发送消息，实现刷新所有客户端的配置。
![bus2](https://gitee.com/clownfish7/image/raw/master/springcloud-bus/bus2.png 'bus2')
###### 触发服务端
  利用消息总线，触发一个服务端 的 /bus/refresh 端点。通过Config Server 服务端向 Bus 总线发送消息，实现刷新所有客户端的配置。
![bus3](https://gitee.com/clownfish7/image/raw/master/springcloud-bus/bus3.png 'bus3')
###### 如何选型
  根据架构图显然 图二 更加合适，所以推荐使用 触发服务端 Config Server 的方式。图一触发客户端方式 不适合的原因如下：
1. 利用消息总线触发客户端方式，打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新的职责；
2. 触发客户端方式，破坏了微服务各个节点之间的对等性（比如说：3355/3366/3377 集群方式提供服务，此时 3355 还需要消息通知，影响节点的对等性）
3. 有一定的局限性。当微服务迁移时，网络地址会经常发生变化，如果此时需要做到自动刷新，则会增加更多的修改。

##### 14.3 SpringCloud Bus 动态刷新全局广播配置
  pom.xml 在服务端配置中心 Config Server、客户端集群 中引入 Bus 总线依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```
  yml 配置 rabbitmq 和暴露 refresh 端点
```yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: username
    password: password
    virtual-host: /
# 服务端
management:
  endpoints:
    web:
      exposure:
        #为什么配置 bus-refresh，看工作原理图
        include: 'bus-refresh'
```
  向 服务端 发送 Post 请求，命令：curl -X POST "http://localhost:3344/actuator/bus-refresh"  
  当向 Config Server 发送 Post 请求后，总线上的各个实例都能够及时 监听和消费 配置的变更。使用广播的方式，真正的实现 一处通知，处处生效。  
  使用 MQ 广播的方式，实现 一处通知，处处生效 的效果。此时我们登陆 Rabbit MQ 客户端，在 Exchanges 模块，就能够看到一个叫做 springCloudBus 的 Topic。  
  Config 客户端都去监听 MQ 中的同一个 topic（默认是 springCloudBus）。当一个服务刷新数据的时候，它会把这个消息放入到 Topic 中，这样其他监听同一 Topic 的服务就能够得到通知，然后去更新自身的配置。

##### 14.4 SpringCloud Bus 动态刷新定点通知配置
  如果需要 差异化通知，并不想进行全局广播，此时就用到了 Bus 的 定点通知 功能。
  此处命令和全局广播有点不同，命令为：http://配置中心IP:配置中心的端口号/actuator/bus-refresh/{destination}
  通过指定 /bus/refresh请求 不再发送到具体的服务实例上，而是发给 Config Server 并通过 {destination} 参数 来指定需要更新配置的服务或实例。

  **{destination}** 参数 = **微服务名** :**端口号**。

#### 15. SpringCloud Stream 消息驱动
  官网：[SpringCloud Stream 官网](https://spring.io/projects/spring-cloud-stream#overview 'SpringCloud Stream 官网')  

  Spring Cloud Stream 是一个 构建消息驱动微服务的框架。应用程序通过 inputs 或者 outputs 来与 Spring Cloud Stream 中的 binder 对象交互。通过我们的配置来进行 binding(绑定)， 然后 Spring Cloud Stream 通过 binder 对象与消息中间件交互。所以，我们只需要搞清楚如何与 Spring Cloud Stream 交互，就可以方便使用消息驱动的方式。

  Spring Cloud Stream 通过使用 Spring Integration 来连接消息代理中间件，以实现消息时间驱动。Spring Cloud Stream 为一些供应商的消息中间件产品提供了个性化的自动配置发现，引用了 发布-订阅、消费组、分区 三个核心概念。目前仅支持 **RabbitMQ**、**Kafka**。

  **一句话总结**： Spring Cloud Stream 屏蔽了底层消息中间件的差异，降低 MQ 切换成本，统一消息的编程模型。开发中使用的就是各种 **xxxBinder**。
##### 15.1 标准MQ 和 Spring Cloud Stream 对比
######  标准 MQ 结构图
  生产者/消费者 之间通过 消息媒介 传递消息内容
![stream1](https://gitee.com/clownfish7/image/raw/master/springcloud-stream/stream1.png 'stream1')

######  Spring Cloud Stream 结构图
  比如说我们用到了 RabbitMQ 和 Kafka，由于这两个消息中间件的架构上的不同。像 RabbitMQ 有 `exchange`，Kafka 有` Topic` 和 `Partions` 分区的概念。
  这些中间件的差异性，给我们实际项目的开发造成了一定的困扰。我们如果用了两个消息队列中的其中一个，后面的业务需求如果向往另外一种消息队列进行迁移，这需求简直是灾难性的。因为它们之间的耦合性过高，导致一大堆东西都要重新推到来做，这时候 Spring Cloud Stream 无疑是一个好的选择，它为我们提供了一种解耦合的方式。

![stream2](https://gitee.com/clownfish7/image/raw/master/springcloud-stream/stream2.png 'stream2')

##### 15.3 Spring Cloud Stream 如何统一底层差异
  在没有绑定器这个概念的情况下，我们的 Spring Boot 应用直接与消息中间件进行信息交互时，由于个消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性。

  通过定义绑定器(Binder)作为中间层，就可以完美的实现应用程序与消息中间件细节的隔离。 通过向应用程序暴露统一的 Channel 通道，使得应用程序不需要在考虑各种不同的消息中间件的实现。

![stream3](https://gitee.com/clownfish7/image/raw/master/springcloud-stream/stream3.png 'stream3')

  默认情况下，RabbitMQ Binder 实现 将每个目标映射到TopicExchange。 对于每个使用者组，队列都绑定到该 TopicExchange。 每个使用者实例在其组的队列中都有一个对应的 RabbitMQ使用者实例。 对于分区的生产者和使用者，队列以分区索引为后缀，并使用分区索引作为路由键。 对于匿名使用者（没有组属性的使用者），将使用自动删除队列（具有随机的唯一名称）。
##### 15.4 Spring Cloud Stream 执行流程
![stream4](https://gitee.com/clownfish7/image/raw/master/springcloud-stream/stream4.png 'stream4')
**说明：**
1. Source/Sink：Source 输入消息，Sink 输出消息
2. Channel：通道，是队列 Queue 的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel 对队列进行配置；
3. Binder：很方便的 连接中间件，屏蔽 MQ 之间的差异
##### 15.5 Spring Cloud Stream 编码API和常用注解
![stream5](https://gitee.com/clownfish7/image/raw/master/springcloud-stream/stream5.png 'stream5')
##### 15.6 Spring Cloud Stream 配置使用  
  本示例选用 RabbitMQ，在不需要任何 RabbitMQ 包依赖的基础上，使用 Spring Cloud Stream 消息驱动来实现消息的发送&接收。
###### 生产者配置
  pom.xml
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```
  application.yml
```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      # 在此处配置要绑定的rabbitmq的服务信息；
      binders:
        defaultRabbit: # 表示定义的名称，用于binding整合(可以自定义名称)
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      # 服务的整合处理
      bindings:
        output: # 这个名字是一个通道的名称
          # 表示要使用的Exchange名称定义
          destination: studyExchange
          # 设置消息类型，本次为json，文本则设置“text/plain”
          content-type: application/json
          default-binder: defaultRabbit
          # 设置要绑定的消息服务的具体设置(需与自定义名称一致)
          # (飘红：Settings->Editor->Inspections->Spring->Spring Boot->Spring Boot application.yml 对勾去掉)
          binder: defaultRabbit
eureka:
  instance:
    hostname: localhost
    #所在主机ip
    ip-address: 127.0.0.1
    #将自己的ip地址注册到Eureka服务中
    prefer-ip-address: true
    #指定实例id
    instance-id: ${spring.application.name}:${server.port}
    #eureka客户端向服务端发送心跳时间间隔，默认30s
    lease-renewal-interval-in-seconds: 30
    #eureka服务端在收到最后一次心跳后的等待时间，超时将删除服务，90s
    lease-expiration-duration-in-seconds: 90
  client:
    # 表示是否注册进eureka,默认为true
    register-with-eureka: true
    # 表示是否从EurekaServer抓取已有注册信息，默认为true，单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
```
  provider
```java
public interface IMessageProvider {
    public String send();
}
/**
 * @author You
 * @create 2020-08-29 22:19
 * @EnableBinding 已包含 @Configuration -> 包含 @Component
 */
@EnableBinding(Source.class)
public class MessageProviderImpl implements IMessageProvider {

    @Autowired
    private MessageChannel output;

    @Override
    public String send() {
        output.send(MessageBuilder.withPayload(UUID.randomUUID().toString()).build());
        return "ok";
    }
}
```
###### 消费者配置
  pom.xml
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```
  application.yml
```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      # 在此处配置要绑定的rabbitmq的服务信息；
      binders:
        defaultRabbit: # 表示定义的名称，用于binding整合(可以自定义名称)
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      # 服务的整合处理
      bindings:
        input: # 这个名字是一个通道的名称
          # 表示要使用的Exchange名称定义
          destination: studyExchange
          # 设置消息类型，本次为json，文本则设置“text/plain”
          content-type: application/json
          default-binder: defaultRabbit
          # 设置要绑定的消息服务的具体设置(需与自定义名称一致)
          # (飘红：Settings->Editor->Inspections->Spring->Spring Boot->Spring Boot application.yml 对勾去掉)
          binder: defaultRabbit

eureka:
  instance:
    hostname: localhost
    #所在主机ip
    ip-address: 127.0.0.1
    #将自己的ip地址注册到Eureka服务中
    prefer-ip-address: true
    #指定实例id
    instance-id: ${spring.application.name}:${server.port}
    #eureka客户端向服务端发送心跳时间间隔，默认30s
    lease-renewal-interval-in-seconds: 30
    #eureka服务端在收到最后一次心跳后的等待时间，超时将删除服务，90s
    lease-expiration-duration-in-seconds: 90
  client:
    # 表示是否注册进eureka,默认为true
    register-with-eureka: true
    # 表示是否从EurekaServer抓取已有注册信息，默认为true，单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
```
  provider
```java
/**
 * @author You
 * @create 2020-08-30 16:56
 * @EnableBinding 已包含 @Configuration -> 包含 @Component
 */
@EnableBinding(Sink.class)
public class MessageListener {

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println(message.getPayload());
    }

}
```
##### 15.7 Spring Cloud Stream 重复消费/持久化问题
###### 重复消费问题
  当集群方式进行消息消费时，就会存在 消息的重复消费问题。比如订单库存相关消息，购物完成库存 -1，消息重复消费就会导致库存不准确问题出现，这显然是不能接受的。
  这是因为没有进行分组的原因，不同组就会出现重复消费；同一组内会发生竞争关系，只有一个可以消费。 如果我们不指定(8802、8803)集群分组信息，它会默认将其当做两个分组来对待。这个时候，如果发送一条消息到 MQ，不同的组就都会收到消息，就会造成消息的重复消费。
  解决方式很简单，只需要用到 Stream 当中 group 属性对消息进行分组即可。将8802、8803分到一个组即可。
```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      # 在此处配置要绑定的rabbitmq的服务信息；
      binders:
        defaultRabbit: # 表示定义的名称，用于binding整合(可以自定义名称)
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      # 服务的整合处理
      bindings:
        input: # 这个名字是一个通道的名称
          # 表示要使用的Exchange名称定义
          destination: studyExchange
          # 设置消息类型，本次为json，文本则设置“text/plain”
          content-type: application/json
          default-binder: defaultRabbit
          # 设置要绑定的消息服务的具体设置(需与自定义名称一致)
          # (飘红：Settings->Editor->Inspections->Spring->Spring Boot->Spring Boot application.yml 对勾去掉)
          binder: defaultRabbit
          # 分组
          group: group1

eureka:
  instance:
    hostname: localhost
    #所在主机ip
    ip-address: 127.0.0.1
    #将自己的ip地址注册到Eureka服务中
    prefer-ip-address: true
    #指定实例id
    instance-id: ${spring.application.name}:${server.port}
    #eureka客户端向服务端发送心跳时间间隔，默认30s
    lease-renewal-interval-in-seconds: 30
    #eureka服务端在收到最后一次心跳后的等待时间，超时将删除服务，90s
    lease-expiration-duration-in-seconds: 90
  client:
    # 表示是否注册进eureka,默认为true
    register-with-eureka: true
    # 表示是否从EurekaServer抓取已有注册信息，默认为true，单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
```
###### 持久化问题
  还是很简单，还是加一个 group 分组属性就行了。所以说 group 分组属性在消息重复消费和消息持久化消费(避免消息丢失)是一个非常重要的属性，推荐你在使用时加上。
#### 16. SpringCloud Sleuth 分布式请求链路追踪
官网：[SpringCloud Sleuth 官网](https://spring.io/projects/spring-cloud-sleuth#overview SpringCloud Sleuth 官网)

  在微服务框架中，一个由客户端发起的请求，在后端系统中会经过多个不同的微服务节点调用，协同操作产生最后的请求结果。每一个前端请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现 高延时 或者 错误，都会引起整个请求最后的失败。  

> Spring Cloud Sleuth 提供了分布式系统中一套完整的服务跟踪的解决方案，并且兼容支持了zipkin，完美的解决了多个微服务之间链路调用的问题。  

**一句话总结**： 就是用来处理服务之间调用关系的。
##### 16.1 SpringCloud Sleuth 调用结构图
![sleuth1](https://gitee.com/clownfish7/image/raw/master/springcloud-sleuth/sleuth1.png 'sleuth1')
##### 16.2 SpringCloud Sleuth 环境准备
  Zipkin 是 Twitter 的一个开源项目，允许开发者收集 Twitter 各个服务上的监控数据，并提供查询接口。
  我们需要先准备一个 Zipkin 环境。Spring Cloud 从F版起已不需要自己构建 Zipkin server了，只需要调用jar包即可。当前使用版本为 H版。我们只需要下载 Zipkin jar包，使用 java -jar xxx的方式启动即可。
  点击链接：https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server ，下载 zipkin-server-2.12.9-exec.jar 。启动就OK了。
  通过 http://loclahost:9411 就能进入到 Zipkin 为我们提供的可视化界面。
##### 16.3 SpringCloud Sleuth 配置使用
  pom.xml
```xml
<!--包含了sleuth+zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```
  applicaiton.yml
```yml
spring:
  zipkin:
    #监控数据要打到 9411 zipkin 上
    base-url: http://localhost:9411
  sleuth:
    sampler:
      #采样率值介于0到1，1则表示全部采集
      probability: 1
```
  我们打开 http://localhost:9411 Zipkin 控制台就可以看到具体服务调用情况。
  点击相对应请求，还可以看到 模块间调用情况、调用耗时 等更详细的信息。点击导航栏中的 依赖 项，还可以查看模块(调用、被调用)的依赖关系等，链路调用关系一目了然。
  

#### 17. SpringCloud Alibaba 入门简介
  官方文档：
+ [Spring Cloud Alibaba官网]('https://spring.io/projects/spring-cloud-alibaba#overview' Spring Cloud Alibaba官网)
+ [Github英文文档]('https://github.com/alibaba/spring-cloud-alibaba' Github英文文档)
+ [Spring英文文档]('https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html' Spring英文文档)
+ [Github中文文档]('https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md' Github中文文档)

##### 17.1 维护模式
  随着 Spring Cloud Netflix 项目进入维护模式（Maintenance Mode），Eureka、Hystrix、Ribbon、Zuul 等项目都进入了维护模式。
>将模块置于 维护模式 意味着 Spring Cloud 团队将不再向该模块添加新功能。我们将修复 block 级别的 bug 和安全性问题，还将考虑并审查社区中的小请求。自 Spring Cloud Greenwich 版本发行(2018.12.12)以来，Spring Cloud 打算继续为这些模块提供至少一年的支持。（摘自：[官网]('https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now' 官网)）

  现在是 2020.7.20 ，Spring Cloud 已经发行 Hoxton SR6 版本。针对 Spring Cloud Netflix 相关模块已经不再提供支持。我们都知道 Spring Cloud 版本迭代算是比较快的，因而出现了很多重大 ISSUE 都还来不及 Fix 就又推出另一个 Release 版本了。进入维护模式意味着：以后一段时间 Spring Cloud Netflix 提供的服务和功能就这么多了，不再开发新的组件和功能了，这显然无法满足接下来微服务的开发要求。
  伴随着 Spring Cloud Netflix 倒下，一手好牌打的稀巴烂，停更的组件自然就需要寻找替代者来继续下去。
>Alibaba 为了能够在微服务领域占据一定的话语权，此时便趁虚而入，于2018.10.31 Spring Cloud Alibaba 正式入驻 Spring Cloud 官方孵化器，并在 Maven Spring Cloud for Alibaba 0.2.0 released。（附：[Spring Cloud Alibaba 官方介绍]('https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md' Spring Cloud Alibaba 官方介绍) ） 

  Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。
  依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

##### 17.2 Spring Cloud Alibaba 主要功能
  Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。
  依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。
1. 服务限流降级： 默认支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
2. 服务注册与发现： 适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
3. 分布式配置管理： 支持分布式系统中的外部化配置，配置更改时自动刷新。
4. 消息驱动能力： 基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
5. 分布式事务： 使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。。
6. 阿里云对象存储： 阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
7. 分布式任务调度： 提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
8. 阿里云短信服务： 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

##### 17.3 Spring Cloud Alibaba 包含组件
1. Sentinel：阿里巴巴开源产品，把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
2. Nacos：阿里巴巴开源产品，一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
3. RocketMQ：Apache RocketMQ™ 基于 Java 的高性能、高吞吐量的分布式消息和流计算平台。
4. Dubbo：Apache Dubbo™ 是一款高性能 Java RPC 框架。
5. Seata：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
6. Alibaba Cloud OSS：阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
7. Alibaba Cloud SchedulerX：阿里中间件团队开发的一款分布式任务调度产品，支持周期性的任务与固定时间点触发任务。
8. Alibaba Cloud SMS：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。
  `Alibaba Cloud OSS`、`Alibaba Cloud SchedulerX`、`Alibaba Cloud SMS` 是阿里云相关的付费业务。
#### 18. SpringCloud Alibaba Nacos 服务注册和配置中心
##### 18.1 什么是 Nacos
  Nacos（Dynamic Naming and Configuration Service）：一个更易于构建云原生应用的动态服务发现，配置管理和服务管理中心。我们可以理解为：Nacos = 服务注册中心 + 配置中心；等价于 Nacos = Eureka + Spring Cloud Config + Spring Cloud Bus。
  Nacos 可以替代 Eureka 来实现 服务注册中心、可以替代 Spring Cloud Config 来实现服务配置中心、可以替代 Spring Cloud Bus 来实现 配置的全局广播。Nacos 是更强调云原生时代支持 “服务治理、服务沉淀、共享、持续发展” 理念的注册中心和配置中心。(附：[Nacos 官网]('https://nacos.io/zh-cn/index.html' Nacos 官网))

##### 18.2 Nacos 安装运行
  从官网下载 Nacos ：[1.3.1 版本下载地址](https://github.com/alibaba/nacos/releases/tag/1.3.1 '1.3.1 版本下载地址')，你也可以选择指定版本下载：[选择指定版本下载](https://github.com/alibaba/nacos/releases '选择指定版本下载')。
  下载完成后，解压缩，直接运行 bin 目录下的 startup.cmd ，即可启动 Nacos 服务，使用的是 8848 端口。
  运行成功后，直接访问 http://localhost:8848/nacos 就可以进入 Nacos 的为我们提供的 web 控制台。用户名、密码默认为 nacos(1.2.0 版本不需要输入密码)，控制台还是挺清新的哈，还提供中文支持。

**docker 启动**[github说明](https://github.com/nacos-group/nacos-docker/blob/master/README_ZH.md 'github说明')
```shell
docker run -d \
--name nacos \
-p 8848:8848 \
-e JVM_XMS=128m \
-e JVM_XMX=128m \
-e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e NACOS_SERVER_IP=127.0.0.1 \
-e MYSQL_SERVICE_HOST=127.0.0.1 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_DB_NAME=nacos_config \
-e MYSQL_SERVICE_USER=username \
-e MYSQL_SERVICE_PASSWORD=password \
docker.io/nacos/nacos-server
```

##### 18.3 Nacos用作服务注册中心
  Nacos 可以替代 Eureka 来作为 服务注册中心。附：[Nacos 服务注册中心官方文档](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/en-us/index.html#_spring_cloud_alibaba_nacos_discovery 'Nacos 服务注册中心官方文档')。接下来就介绍 Nacos 用作服务注册中心。
  pom.xml 启动类加注解 `@EnableDiscoveryClient`
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```
  application.yml
```yml
server:
  port: 83

spring:
  application:
    name: cloud-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 47.97.8.7:8848

management:
  endpoints:
    web:
      exposure:
        include: "*"
```
  **nacos 默认集成了 ribbon**实现负载均衡，restTemplate切莫忘记加@LoadBalanced

|   | Nacos | Eureka | Consul | CoreDNS | ZooKeeper |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 一致性检查 | CP+AP | AP | CP | / | CP|
| 健康检查 | TCP/HTTP/MySQL/Client Beat | Client Beat | TCP/HTTP/gRPC/Cmd | / | Client Beat|
| 负载均衡 | 权重/DSL/metadata/CMDB | Ribbon | Fabio | RR | / |
| 雪崩保护 | 支持 | 支持 | 不支持 | 不支持 | 不支持 |
| 自动注销实例 | 支持 | 支持 | 不支持 | 不支持 | 支持 |
| 访问协议 | HTTP/DNS/UDP | HTTP | HTTP/DNS | DNS | TCP |
| 监听支持 | 支持 | 支持 | 支持 | 不支持 | 支持 |
| 多数据中心 | 支持 | 支持 | 支持 | 不支持 | 不支持 |
| 跨注册中心 | 支持 | 不支持 | 支持 | 不支持 | 不支持 |
| SpringCloud 集成 | 支持 | 支持 | 支持 | 不支持 | 不支持 |
| Dubbo 集成 | 支持 | 不支持 | 不支持 | 不支持 | 支持 |
| K8s 集成 | 支持 | 不支持 | 支持 | 支持 | 不支持 |

###### Ⅰ Nacos支持AP和CP模式的切换
`C`**一致性** `A` **高可用** `P` **容错性**。参考：[CAP原则](https://baike.baidu.com/item/CAP%E5%8E%9F%E5%88%99/5712863?fr=aladdin 'CAP原则')，主流选用的都是 AP 模式，保证系统的高可用。
###### Ⅱ 何时选择使用何种模式
  一般来说，如果不需要存储服务级别的信息，且服务实例是通过 Nacos-client 注册，并能够保证心跳上报，那么就可以选择 AP 模式。当前主流的服务如 Spring Cloud 和 Dubbo 服务，都适用于 AP 模式，AP模式为了服务的可用行而减弱了一致性，因此 AP 模式下只支持注册临时实例。

  如果需要在服务级别编辑或者存储配置信息，那么 CP 是必须的，K8S服务和 DNS服务则适用于 CP 模式。CP模式下则支持注册持久化实例，此时则是以 Raft 协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。
###### Ⅲ Nacos AP/CP 模式切换命令
  Nacos 集群默认支持的是CAP原则中的 AP原则，但是 也可切换为CP原则，切换命令如下：
```shell
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'
```
  同时微服务的 bootstrap.properties 需配置如下选项指明注册为临时/永久实例，AP模式不支持数据一致性，所以只支持服务注册的临时实例，CP模式支持服务注册的永久实例，满足配置文件的一致性。
```yml
#false为永久实例，true表示临时实例开启，注册为临时实例
spring: 
  cloud: 
    nacos: 
	  discovery: 
		ephemeral: false
```
##### 18.4 Nacos用作服务配置中心
  Nacos 可以替代 Config + Bus 来用作 `服务配置中心`。附：[Nacos服务配置中心官方文档](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/en-us/index.html#_spring_cloud_alibaba_nacos_config 'Nacos服务配置中心官方文档')。Nacos 用作服务配置中心，分为 `基础配置(简单使用)` 和 `分类配置(多环境使用 dev、test、prod等环境)`。接下来就介绍 Nacos 用作服务配置中心。附：[Github Wiki](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config 'Github Wiki')
###### Nacos 基础配置
  pom.xml 主启动类添加开启 @EnableDiscoveryClient
```xml
<dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
```
  application.yml
```yaml
spring:
  profiles:
    active: test
```
  bootstrap.yml
```yaml
server:
  port: 3377

spring:
  application:
    name: cloud-config-client
  cloud:
    nacos:
      config:
        # Nacos 服务配置中心地址
        server-addr: 47.97.8.7:8848
        # 指定 yaml 格式的配置
        file-extension: yaml
        # Nacos 命名空间
        namespace: 8a6827d8-4c11-40a6-9efd-0d92f1bb830e
        # Nacos 分组
        group: group1
        # Nacos 集群 cluster
        cluster-name: cluster1
      discovery:
        # Nacos 服务配置中心地址
        server-addr: 47.97.8.7:8848
        # Nacos 命名空间
        namespace: 8a6827d8-4c11-40a6-9efd-0d92f1bb830e
        # Nacos 分组
        group: group1
        # Nacos 集群 cluster
        cluster-name: cluster1
```
  controller.java 业务类添加 @RefreshScope 实现配置自动更新
```java
@RefreshScope
@RestController //通过Spring Cloud 原生注解 @RefreshScope 实现配置自动更新
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```
  进入 Nacos → 配置管理 → 配置列表 → + 号 添加配置项。Data ID 按规则编写，Group 在接下来的 分类配置 会有介绍。
**Nacos 自带动态刷新**
  使用 Spring Cloud Config 时，需要配合 Spring Cloud Bus + RabbitMQ 中间件，进行广播方式才能 实现动态刷新。Nacos则自带动态刷新,修改下Nacos中的yaml配置文件，再次调用查看配置的接口，就会发现配置已经刷新。好用！！！
**dataId 命名规则**
  在 Nacos Spring Cloud 中， `dataId` 有明确的配置规则，官方也有说明。进入链接查看：[官网链接](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html '官网链接')。此处也做一点简单介绍。`dataId` 的完整格式如下：
```
${prefix}-${spring.profile.active}.${file-extension}
```
+ `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
+ `spring.profile.active` 即为当前环境对应的 `profile`。注意：当 `spring.profile.active` 为空时，对应的连接符 `-` 也将不存在，`dataId` 的拼接格式变成 `${prefix}.${file-extension}`（建议：不要让 `spring.profile.active` 为空，或许会有一些意外的问题）
+ `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。**（此处yaml与yml有区别）**
###### Nacos 分类配置
  项目开发中，一定会遇到 `多环境`、`多项目管理` 问题。遇到下面问题时，Nacos 基础配置显然无法解决这些问题，接下来就对 `Nacos 命名空间` 及 `Group` 相关概念的了解。
  问题1： 实际开发中，通常一个系统会准备 `dev开发环境`、`test测试环境`、`prod生产环境`，如何保证指定环境启动时服务能够正确读取到 Nacos 上相应环境的配置文件？
  问题2： 一个大型的分布式微服务系统会有很多个微服务子项目，每个微服务项目又都会有相应的 `dev开发环境`、`test测试环境`、`prod生产环境` 等，那怎么对这些微服务配置进行管理呢？

**Ⅰ. Nacos的命名规则说明**
  Nacos 命名由 `Namespace(命名空间)` + `Group(分组)` + `Data ID(实例ID)` 三部分组成，类似于 Java 中的 `package(报名)` + `class(类名)` 方式。最外层 `Namespace` 用于区分部署环境；`Group` 和 `Data ID` 逻辑上用于区分两个目标对象。
  默认情况下：Namespace = public，Group = DEFAULT_GROUP，Cluster=DEFAULT
  `Namespace` 主要用来实现隔离，Nacos 默认的命名空间是 public。比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个 Namespace，不同的 Namespace 之间是隔离的；
  `Group` 是一组配置集，是组织配置的维度之一。默认是 DEFAULT_GROUP。通过一个有意义的 名称 对配置集进行分组，从而区分 Data ID 相同的配置集。配置分组的常见场景：不同的应用或组件使用了相同的配置类型，就可以把不同的微服务划分到同一个分组里面去，从而解决问题2；如 database_url 配置和 MQ_topic 配置。
  `Service` 微服务；一个 Service 可以包含多个 Cluster(集群)，Nacos 默认 Cluster 是 DEFAULT，Cluster 是对指定微服务的一个虚拟划分。比方说为了容灾，将 Service 微服务分别部署在了杭州机房和广州机房，这是就可以给杭州机房的 Service 微服务起一个集群名称(HZ)，给广州机房的 Service 微服务起一个集群名称(GZ)，还可以尽量让同一个机房的微服务互相调用，以提升性能。
  `Instance`，就是一个个微服务实例。
**Ⅱ. 新建Namespace**
  选择 命名空间 → 新建命名空间，进行命名空间的设置。在 Nacos 1.1.4 版本，还不支持自定义命名空间ID，Nacos 1.2.0 版本后开始支持自定义命名空间ID 了。更推荐你使用自定义命名空间。

#### 19. SpringCloud Alibaba Sentinel 熔断与限流
1. [Sentinel GitHub 官网](https://github.com/alibaba/Sentinel 'Sentinel GitHub 官网')
2. [Sentinel Wiki中文介绍文档](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D 'Sentinel Wiki中文介绍文档')
3. [Spring Cloud 关于 Sentinel 使用文档](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/en-us/index.html#_spring_cloud_alibaba_sentinel 'Spring Cloud 关于 Sentinel 使用文档')
##### 19.1 Sentinel 是什么
  随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
Sentinel 具有以下特征:

+ 丰富的应用场景：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
+ 完备的实时监控：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
+ 广泛的开源生态：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
+ 完善的 SPI 扩展点：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

Sentinel 分为两个部分:
+ 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
+ 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。
#### 20. SpringCloud Alibaba Seata 处理分布式事务
1. [Seata 官网](http://seata.io/zh-cn/index.html 'Seata 官网')
2. [Seata 源码 GitHub 地址](https://github.com/seata/seata 'Seata 源码 GitHub 地址')
3. [Seata 官方文档(比较鸡肋，介绍不清楚)](http://seata.io/zh-cn/docs/overview/what-is-seata.html 'Seata 官方文档')
4. [Seata 下载地址](http://seata.io/zh-cn/blog/download.html 'Seata 下载地址')
##### 20.1 分布式事务的问题
  学习到Seata，我默认大家对事务已经有了一定的了解，关于事务此处就不过多介绍。在之前 单机单库 环境下，针对事务的处理还是比较简单的。尤其是结合 Spring 框架，可以说是一个@Transaction 注解走天下。事务 & Spring 事务相关内容，点击链接去了解吧：[事务 & Spring 事务内容介绍](https://blog.csdn.net/lzb348110175/article/details/104854696 '事务 & Spring 事务内容介绍')
  在如今 Spring Cloud 分布式微服务架构体系中，按业务模块划分，一个模块使用一个数据库。多个模块配合来完成一个业务，官网微服务实例 [官网](http://seata.io/zh-cn/docs/user/quickstart.html '官网')。
  单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用三个独立的数据源，业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证，但是全局数据一致性问题是无法保证的。救世主 Seata 它来了。

##### 20.2 什么是 Seata
  `Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。` Seata 将为用户提供了 `AT`、`TCC`、`SAGA` 和 `XA` 事务模式，为用户打造一站式的分布式解决方案。
  在 Seata 开源之前，Seata 对应的内部版本在阿里经济体内部一直扮演着分布式一致性中间件的角色，帮助经济体平稳的度过历年的双11，对各部门业务进行了有力的支撑。经过多年沉淀与积累，商业化产品先后在阿里云、金融云进行售卖。2019.1 为了打造更加完善的技术生态和普惠技术成果，Seata 正式宣布对外开源，未来 Seata 将以社区共建的形式帮助其技术更加可靠与完备。

```shell
docker run -d \
--name seata \
-p 8091:8091 \
-e SEATA_IP=47.97.8.7 \
-e SEATA_PORT=8091 \
-e SEATA_CONFIG_NAME=file:/root/seata-config/registry \
-v /data/seata/config:/root/seata-config  \
seataio/seata-server
```
file.json
```json
transport {
  # tcp udt unix-domain-socket
  type = "TCP"
  #NIO NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  # the client batch send request enable
  enableClientBatchSendRequest = false
  #thread factory for netty
  threadFactory {
    bossThreadPrefix = "NettyBoss"
    workerThreadPrefix = "NettyServerNIOWorker"
    serverExecutorThreadPrefix = "NettyServerBizHandler"
    shareBossWorker = false
    clientSelectorThreadPrefix = "NettyClientSelector"
    clientSelectorThreadSize = 1
    clientWorkerThreadPrefix = "NettyClientWorkerThread"
    # netty boss thread size,will not be used for UDT
    bossThreadSize = 1
    #auto default pin or 8
    workerThreadSize = "default"
  }
  shutdown {
    # when destroy server, wait seconds
    wait = 3
  }
  serialization = "seata"
  compressor = "none"
}

## transaction log store, only used in server side
store {
  ## store mode: file、db
  mode = "db"
  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    maxBranchSessionSize = 16384
    # globe session size , if exceeded throws exceptions
    maxGlobalSessionSize = 512
    # file buffer size , if exceeded allocate new buffer
    fileWriteBufferCacheSize = 16384
    # when recover batch read size
    sessionReloadReadSize = 100
    # async, sync
    flushDiskMode = async
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://47.97.8.7:3306/seata"
    user = "dev"
    password = "productdev123"
    minConn = 5
    maxConn = 30
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
  }
}
## server configuration, only used in server side
server {
  recovery {
    #schedule committing retry period in milliseconds
    committingRetryPeriod = 1000
    #schedule asyn committing retry period in milliseconds
    asynCommittingRetryPeriod = 1000
    #schedule rollbacking retry period in milliseconds
    rollbackingRetryPeriod = 1000
    #schedule timeout retry period in milliseconds
    timeoutRetryPeriod = 1000
  }
  undo {
    logSaveDays = 7
    #schedule delete expired undo_log in milliseconds
    logDeletePeriod = 86400000
  }
  #check auth
  enableCheckAuth = true
  #unit ms,s,m,h,d represents milliseconds, seconds, minutes, hours, days, default permanent
  maxCommitRetryTimeout = "-1"
  maxRollbackRetryTimeout = "-1"
  rollbackRetryTimeoutUnlockEnable = false
}

## metrics configuration, only used in server side
metrics {
  enabled = false
  registryType = "compact"
  # multi exporters use comma divided
  exporterList = "prometheus"
  exporterPrometheusPort = 9898
}

```
registry.conf
```json
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "47.97.8.7:8848"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = ""
    password = ""
  }
  eureka {
    serviceUrl = "http://localhost:8761/eureka"
    application = "default"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = 0
    password = ""
    cluster = "default"
    timeout = 0
  }
  zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  consul {
    cluster = "default"
    serverAddr = "127.0.0.1:8500"
  }
  etcd3 {
    cluster = "default"
    serverAddr = "http://localhost:2379"
  }
  sofa {
    serverAddr = "127.0.0.1:9603"
    application = "default"
    region = "DEFAULT_ZONE"
    datacenter = "DefaultDataCenter"
    cluster = "default"
    group = "SEATA_GROUP"
    addressWaitTime = "3000"
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    serverAddr = "47.97.8.7:8848"
    namespace = ""
    group = "SEATA_GROUP"
    username = ""
    password = ""
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  apollo {
    appId = "seata-server"
    apolloMeta = "http://192.168.1.204:8801"
    namespace = "application"
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  file {
    name = "file.conf"
  }
}

```
config.txt
```txt
transport.type=TCP
transport.server=NIO
transport.heartbeat=true
transport.enableClientBatchSendRequest=false
transport.threadFactory.bossThreadPrefix=NettyBoss
transport.threadFactory.workerThreadPrefix=NettyServerNIOWorker
transport.threadFactory.serverExecutorThreadPrefix=NettyServerBizHandler
transport.threadFactory.shareBossWorker=false
transport.threadFactory.clientSelectorThreadPrefix=NettyClientSelector
transport.threadFactory.clientSelectorThreadSize=1
transport.threadFactory.clientWorkerThreadPrefix=NettyClientWorkerThread
transport.threadFactory.bossThreadSize=1
transport.threadFactory.workerThreadSize=default
transport.shutdown.wait=3
service.vgroupMapping.my_test_tx_group=default
service.default.grouplist=47.97.8.7:8091
service.enableDegrade=false
service.disableGlobalTransaction=false
client.rm.asyncCommitBufferLimit=10000
client.rm.lock.retryInterval=10
client.rm.lock.retryTimes=30
client.rm.lock.retryPolicyBranchRollbackOnConflict=true
client.rm.reportRetryCount=5
client.rm.tableMetaCheckEnable=false
client.rm.sqlParserType=druid
client.rm.reportSuccessEnable=false
client.rm.sagaBranchRegisterEnable=false
client.tm.commitRetryCount=5
client.tm.rollbackRetryCount=5
store.mode=db
store.file.dir=file_store/data
store.file.maxBranchSessionSize=16384
store.file.maxGlobalSessionSize=512
store.file.fileWriteBufferCacheSize=16384
store.file.flushDiskMode=async
store.file.sessionReloadReadSize=100
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://47.97.8.7:3306/seata?useUnicode=true
store.db.user=dev
store.db.password=productdev123
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
client.undo.dataValidation=true
client.undo.logSerialization=jackson
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000
client.undo.logTable=undo_log
client.log.exceptionRate=100
transport.serialization=seata
transport.compressor=none
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
```
nacos-config.sh
```shell
#!/usr/bin/env bash
# Copyright 1999-2019 Seata.io Group.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at、
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

while getopts ":h:p:g:t:" opt
do
  case $opt in
  h)
    host=$OPTARG
    ;;
  p)
    port=$OPTARG
    ;;
  g)
    group=$OPTARG
    ;;
  t)
    tenant=$OPTARG
    ;;
  ?)
    echo " USAGE OPTION: $0 [-h host] [-p port] [-g group] [-t tenant] "
    exit 1
    ;;
  esac
done

if [[ -z ${host} ]]; then
    host=localhost
fi
if [[ -z ${port} ]]; then
    port=8848
fi
if [[ -z ${group} ]]; then
    group="SEATA_GROUP"
fi
if [[ -z ${tenant} ]]; then
    tenant=""
fi

nacosAddr=$host:$port
contentType="content-type:application/json;charset=UTF-8"

echo "set nacosAddr=$nacosAddr"
echo "set group=$group"

failCount=0
tempLog=$(mktemp -u)
function addConfig() {
  curl -X POST -H "${1}" "http://$2/nacos/v1/cs/configs?dataId=$3&group=$group&content=$4&tenant=$tenant" >"${tempLog}" 2>/dev/null
  if [[ -z $(cat "${tempLog}") ]]; then
    echo " Please check the cluster status. "
    exit 1
  fi
  if [[ $(cat "${tempLog}") =~ "true" ]]; then
    echo "Set $3=$4 successfully "
  else
    echo "Set $3=$4 failure "
    (( failCount++ ))
  fi
}

count=0
for line in $(cat $PWD/config.txt | sed s/[[:space:]]//g); do
  (( count++ ))
	key=${line%%=*}
  value=${line#*=}
	addConfig "${contentType}" "${nacosAddr}" "${key}" "${value}"
done

echo "========================================================================="
echo " Complete initialization parameters,  total-count:$count ,  failure-count:$failCount "
echo "========================================================================="

if [[ ${failCount} -eq 0 ]]; then
	echo " Init nacos config finished, please start seata-server. "
else
	echo " init nacos config fail. "
fi

```
pom.xml @EnableDiscoveryClient @GlobalTransactional
```xml
<dependenices>
    <!-- nacos -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!-- seata -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    </dependency>
</dependenices>
```

```java
```

##### 20.3 AT 模式
##### 20.4 TCC 模式
##### 20.5 SAGA 模式
##### 20.6 XA 模式