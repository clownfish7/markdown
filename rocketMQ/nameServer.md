

### 1. NameServer 启动流程

nameServer 启动类 NameServerStartup

Step1：解析配置文件、填充 NameServerConfig 

Step2：根据启动属性创建 NamesrvController 并初始化该实例，这是 NameServer 核心控制器
加载 KV 配置，创建 NettyServer 网络处理对象，然后开启两个定时任务，在 RocketMQ
中此类定时任务统称为心跳检测
定时任务 1: NameServer 每隔 Os 扫描一次 Broker 移除处于不激活状态的 Broker
定时任务 2: NameServer 每隔 10 分钟打印一次 KV 配置

Step3：注册 JVM 钩子函数并启动该服务，以便监听 Broker、消息生产者的网络请求




### 2. NameServer 路由注册、故障剔除
    NameServer 主要作用是为消息生产者和消费者提供关于主题 Topic 的路由信息，那么 NameServer 需要、存储路由的基础信息，还要能够管理 Broker 节点，包括路由注册、删除功能

#### 2.1 路由元信息

```java
class RouteInfoManager{
    private final HashMap<String, List<QueueData>>  topciQueueTable;
    private final HashMap<String, BrokerData>       brokerAddrTable;
    private final HashMap<String, Set<String>>      clusterAddrTable;
    private final HashMap<String, BrokerLiveInfo>   brokerLiveTable;
    private final HashMap<String, List<String>>     filterServerTable;
}
```

#### 2.2 路由注册
Step1：RocketMQ 路由注册是通过 BrokerNameServer 的心跳功能实现的, Broker 启动时集群中所有的 NameServ 发送心跳语句，
每隔 30s 集群中所有 NameServer 发送心跳包， NameServer 收到 Broke 心跳包时会更新 brokerLiveTab 缓存中 BrokerLivelInfo
LastUpdateTimestamp ，然后 NameServer 每隔 10s 扫描 brokerLiveTable ，如果连续 120s
没有收到心跳包， NameServ 将移除该 Broker 的路由信息并同时关闭 Socket 连接

Step1：解析配置文件、填充 NameServerConfig

Step1：解析配置文件、填充 NameServerConfig

Step1：解析配置文件、填充 NameServerConfig

Step1：解析配置文件、填充 NameServerConfig

Step1：解析配置文件、填充 NameServerConfig

Step1：解析配置文件、填充 NameServerConfig

Step1：解析配置文件、填充 NameServerConfig

Step1：解析配置文件、填充 NameServerConfig 