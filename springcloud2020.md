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
&emsp; Springboot 启动类
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
&emsp; xxxxxxxxxxxxxxxxxxx

#### 7. Consul 服务注册与发现
&emsp; xxxxxxxxxxxxxxxxxxx

#### 8. Ribbon 负载均衡服务调用
&emsp; xxxxxxxxxxxxxxxxxxx

#### 9. OpenFeign 服务接口调用
&emsp; xxxxxxxxxxxxxxxxxxx

#### 10. Hystrix 断路器
&emsp; xxxxxxxxxxxxxxxxxxx

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

#### 17. SpringCloud Alibaba 入门简洁
&emsp; xxxxxxxxxxxxxxxxxxx

#### 18. SpringCloud Alibaba Nacos 服务注册和配置中心
&emsp; xxxxxxxxxxxxxxxxxxx

#### 19. SpringCloud Alibaba Sentinel 熔断与限流
&emsp; xxxxxxxxxxxxxxxxxxx

#### 20. SpringCloud Alibaba Seata 处理分布式事务
&emsp; xxxxxxxxxxxxxxxxxxx