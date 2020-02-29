## 高可用架构设计启示    
[TOC]
### 一、高可用定义
1. 高可用定义
   高可用：High Availability，简称HA，通过设计减少系统不能提供服务的时间。假设系统一直能够提供服务，我们说系统的可用性是100%。
   
2. 可用性公式   
   **可用性 = 系统正常可用的时长/系统运行总时长 * 100%**   
   或：    
   **可用性 = （系统运行总时长-系统不可用时长/系统运行时长） * 100%**
3. 可用性目标    
   高可用理想值是：100%    
    2个9：99%    
    3个9：99.9%    
    4个9：99.99%

### 二、高可用因子

1. 硬件：机器老化，CPU、内存、硬盘故障或突发断电
2. 网络：网络抖动，网络故障
3. 软件：程序代码BUG，导致突发故障
4. 流量：流量暴增，例如，突发热门流量
5. 容量：容量爆满，例如内存，cpu吃紧，redis内存爆满
6. 依赖：依赖的服务不可用

### 三、高可用策略

- 策略一：扩展实现冗余    
  按照实际情况评估资源，使用**2倍以上**的资源，即使垮掉一半也能完全支撑业务。    
  1. 部署机房：双或者多中心机房部署，热备方式部署    
  2. 应用架构：支持横向分布式扩展，支持多中心部署    
  3. 数据库层：一主多从、读写分离、双集群部署

- 策略二：故障自动转移
  1. 客户端：提供多组域名，重试和ACK机制自动切       
  2. 代理层：重试或探测自动切   
  例如：nginx自动切、backup使用主备，多个Nginx通过keepalived存活探测    
  3. 服务层：自动剔除下线，    
   例如：调用服务发现(Eureka、Dubbo)检查自动剔除   
   又如：Eureka 分区（Zone）实现跨机房容灾调用    
  4. 缓存层：实现主从切换    
   例如：redis-sentinal哨兵机制监控主节点是否正常    
  5. 数据层：主从切换，双集群同步    
   例如：mysql主从切换MHA方案，Consul监控并做主从切换   
   又如：双集群数据通过otter实现同步，故障时流量自动切正常的一方    
  6. MQ层：主从切换   
   例如：Kafka的每个分区Partition可以有多个副本Replication,且多个Replication分步在不同的Broker，其中有一个Leader和多个Follower，当Leader挂掉是，会从同步状态的分区副本isr(in-sync-replica)中重新选择一个Replication作为Leader。

- 策略三：限流降级隔离熔断   
  1. 限流 计数器、令牌桶、漏桶    
  业务层：一般用Google Guava的RateLimiter（令牌桶），分布式情况下需要用redis或zookeeper等作为锁    
  代理层：Tengine/Nginx限流，Sentinel、Spring Cloud Gateway、Zuul 网关限流    
  2. 隔离熔断    
   Hystrix通过线程池隔离、信号量方式隔离，根据配置参数执行熔断     
  3. 降级
  降级方式：自动执行和手动配置    
  例如：设置超时时长降级，例如调用头像服务100秒超时,熔断方式降级等。    
  又如： 通过配置中心下发命名降级  

  实际业务场景：    
  例如：聊天弹幕太多，降级为仅收到自己发的信息（自嗨）；    
  又如：用户头像服务挂了，调用通过配置不下发头像，但聊天功能可用    
  

- 策略四：资源弹性伸缩     
  docker+k8s部署方式，实现自动扩容，当出现大流量或者故障情况性，可以及时自动按规则进行扩容。


- 策略五：定期故障演练     
  知道演练计划，对基础模块和核心业务模块进行故障演练，模拟部分机器、单个机房出现网络故障，检查系统的高可用性，不断优化系统。

- 策略六：完善监控告警    
  使用完整的APM系统进行监控，做到**自动告警**。
  1. 硬件层监控：监控运行中服务器硬件状态   
  2. 网络层监控：监控网络状态和丢包情况   
  3. 客户端监控：客户端增加埋点，自动上报使用异常情况   
  4. 代理层监控：网关、nginx    
  5. 基础中间件：服务注册中心、配置中心、消息队列、分布式缓存、分布式数据库、日志中心（ELK)    
  6. 业务层监控：服务间调用链（Zipkin、SkyWalking、PingPoint）、服务内存和CPU占用情况、GC情况、系统负载均衡情况，接口QPS和延迟，系统错误日志   

- 策略七：保证代码质量   
  
  1. 遵守规范    
  2. 质量扫描    
  3. 充分测试    
  4. 代码审核

- 策略八：遵守上线规范    
  对线上服务和数据保持**敬畏之心**，严格遵守操作流程    
  1. 准备充分：制定上线计划，准备好checklist，提前通知有关人员   
  2. 灰度发布：指定灰度计划，更新局部验收确认后全量。    
  3. 回滚止损：发现问题及时回滚，降低对业务的损失。     
  4. 验收充分：验收过程要仔细严谨，不能漏掉细节。
  

  



  



