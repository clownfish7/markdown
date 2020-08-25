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
&emsp; xxxxxxxxxxxxxxxxxxx

#### 4. 微服务架构编码构建
&emsp; 约定 > 配置 > 编码  
maven下载不了 -> <https://blog.csdn.net/HeyWeCome/article/details/104543411>

#### 5. Eureka 服务注册与发现
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

#### 8. Ribbon 负载均衡服务调用
&emsp; xxxxxxxxxxxxxxxxxxx
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

#### 9. OpenFeign 服务接口调用
&emsp; xxxxxxxxxxxxxxxxxxx
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
&emsp; 默认配置：com.netflix.hystrix.HystrixCommandProperties

![head](https://gitee.com/clownfish7/image/raw/master/head/head.jpg 'head')

![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png '123')

##### 10.1 服务降级
##### 10.2 服务熔断
##### 10.3 服务限流

#### 11. Zuul 网关
&emsp; xxxxxxxxxxxxxxxxxxx

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