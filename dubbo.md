+ Service
  + 业务层
+ Config
  + 配置层 围绕ServiceConfig、ReferenceConfig为中心的配置层
+ Proxy
  + 服务代理层 服务接口透明代理，以ServiceProxy为中心，扩展接口为ProxyFactory
+ Registry
  + 注册中心层 封装服务的注册与发现，以URL为中心，扩展接口为RegistryFactory，Registry，RegistryService
+ Cluster
  + 路由层 封装多个提供者的路由及负载均衡，以Invoker为中心，扩展接口为Cluster、Directory、Router、LoadBalance
+ Monitor
  + RPC调用次数和时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory、Monitor、MonitorService
+ Protocol
  + 远程调用层 封装RPC调用，以Invoker、Result为中心，扩展接口为 Protocol、Invoker、Exporter
+ Exchange
  + 信息交换层 封装请求响应模式，以Request、Response为中心，扩展接口为Exchanger、ExchangeChannel、ExchangeClient、ExchangeServer
+ Transport
  + 网络传输层 抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel、Transporter、Client、Server、Codec
+ Serialize
  + 数据序列化层 可复用的一些工具，扩展接口为 Serialization、ObjectInput、ObjectOutput，ThreadPool



