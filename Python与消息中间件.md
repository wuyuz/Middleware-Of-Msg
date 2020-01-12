## Python与消息中间件



#### 消息中间件概念：

- 什么时消息中间件 (消息队列MQ)：

  队列的主要结构： 生产者、消息队列、消费者

  ![1578792486091](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578792486091.png)



- **消息中间件应用场景**： 异步通信、应用解耦、缓解流量高峰压力

  异步通信：就是将比较耗时的、不需要马上返回结果的操作作为消息存入消息队列中，实现应用层的解耦

  应用解耦：两个对象、程序中存在依赖关系；解耦合就是减少连个模块中的依赖关系，结合消息中间件就是生产者和消费者都不需要关注对方的状态，唯一的关心点就是数据格式统一，且各自的接口只提供给消息中间件使用

  缓解流量高峰压力：如当一个时间点流量的峰值大于系统的并发峰值时，可以存入消息中间件中，而不影响业务

- 分布式系统要求中间件的三个条件：高性能（HIgh Performance）、高可用（High Availability）、横向扩展(Scale Out)

  高性能：单机处理能力强，即每秒可读写的次数多；

  高可用：就是7*24小时不间断服务，正常工作；主要体现在系统某个节点故障后，系统仍然可用，不会丢失数据

  横向可扩展：指增加服务器数量，来提示系统性能；纵向可扩展性就是提升单台服务器的性能，但是纵向可扩展性很容易局限

  

#### 常见消息中间件比较

- Redis: 轻量级、通常嵌入模块内部使用，缓存一些数据小的数据
- RabbitMQ: 健壮、稳定、易用、跨平台、支持多种语言、文档齐全有消息确认机制和持久化机制，可靠性高  
- RocketMQ:  RocketMQ出自阿里的开源产品 ， 支持多种消费模式，包括集群消费、广播消费等 
- ZeroMQ: 号称最快的消息队列系统，专门为高吞吐量/低延迟的场景开发 
- Kafka:  高性能、稳定以及消费者可重复消费消息，常用作大型系统的数据中转站



#### 选择消息中间件时要考虑的因素：

- 实现语言: 擅长的技术树，如果数据源代码，可迅速排bug
- 对外接口：接口的友好型，可增加开发效率
- 持久化策略：消息的存储方式、消息队列崩溃后是否丢失数据有关
- 消息处理模式：主要是消费者是轮询消息，还是主动push给消费者
- 时序保证：消费者获取消息时能否保证消息的顺序，在有关事务的场景下，如金融、转账等时序的保证很重要

![1578835849155](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578835849155.png)



#### 消息中间件的概念和角色

- producer/broker/consumer： 注意：broker主要出现在kafka中可以理解为一个服务器，有多个队列
- queue/channel/topic  : 存储消息的队列
- partition
- publish/subscribe
- Acknowledge

![1578836179170](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578836179170.png)

注意在kafka中消息队列topic由多个partition组成，它是有序的但是topic不能完全保证有序,为了保证负载均衡通常采用哈希算法/run lubi（轮转）算法写入数据到topic中的partition中，同时每个broker都有副本保存数据，如果有一个意外下线，可通过副本继续工作:

![1578836337550](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578836337550.png)



#### 常用机制：

![1578836693012](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578836693012.png)



#### Redis部分

![1578836777768](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578836777768.png)

lua扩展：可自己编写脚本，定制自己的redis，如修改redis的命令



#### 五大数据类型：

- string ： 
  - 缓存二进制数据、如图片，序列化数据等
  - 计数器，如文章访问量
  - 位运算，节约内存
- list：
  - 获取最新的N条数据
  - 消息队列,（由于其是双向链表，两边都可操作，一边放一边取）
  - 实时分析系统，比如：服务器监控程序（将服务器ip存入队列中，通过rpop()、lpush() 可实现队列传送带来回传递数据）
- hash：
  - 存储具有多个属性的对象，比如用户的年龄、姓名、积分等（类似于字典）
- set：
  - 无序，不重复、唯一性集合之间的运算
  - 如通过交集实现共同关注，共同好友等

- sorted set：
  - 有序集合、唯一性、提供分数来进行排序



#### Kafka部分

![1578838068648](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578838068648.png)



#### Kafka的生产者特性：

- 异步通信，所有网络请求异步发送
- 批量发送，通过设置batch size或者timeout一次发送多个消息
- 线程安全，多个线程之间可以共享单个生产者实例
- 负载均衡，采用内部默认机制或者自定义负载均衡策略
- 返回结果，返回消息的topic等元素



#### Kafka的消费者特性：

- 统一的API，不再区分high-level/low-level
- 多次消费，不会删除以消费的消息，只要设置offset，这就支持离线数据了
- 负载均衡，基于partition和consumer group自动负载均衡，每个消费者都有一个组id，同一个组id会被视为同一个组，使得将组绑定到topic中实现负载均衡
- 流量控制：允许开发者控制每次请求返回的消息条数



#### Kafka安全特性：

- 连接认证：连接到服务器的生产者和消费者客户端使用SSL/SASL进行验证
- 权限管理，broker连接Zookeeper进行权限管理
- 加密传输，数据传输进行加密
- 授权管理，客户端读写可以进行授权管理



#### Kafka连接器

![1578839071500](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578839071500.png)



#### Kafka的应用场景：

海量的吞吐量、工业级的稳定性

- 消息服务器
- 网址活动跟踪
- 实时数据流聚合
- 日志聚合



#### Kafka安装配置：

 https://www.bilibili.com/video/av50435278?p=9 

注意：broker.id 为kafka在集群中的唯一标识

可在配置文件中，配置文件的最大数量、存储时长



#### Kafka 相关概念：

- Cluster：集群
- Broker：集群中的一个服务器
- Producer：生产者
- Consumer：消费者
- Consumer Group：多个消费者组，实现topic广播 



#### Kafka消息概念：

- Record：消息的所有元数据
- Topic：消息类别，可以理解为队列
- Partition：分区，每个Topic至少有一个Partition，它是有序的
- Segment：每个Partition有一个或多个Segment，用于存放数据
- Offset： 唯一标识一条消息
- Replication：副本，支持Partition副本
- Leader：topic中会选出一个partition来处理读写请求 

![1578840798836](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578840798836.png)

![1578840944346](C:\Users\wanglixing\Desktop\知识点复习\Django\gif\1578840944346.png)





#### Python使用Kafka开发

视频： https://www.bilibili.com/video/av50435278?p=11 

