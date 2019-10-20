## *Spring Boot 2.0*
	Webflux 是一个全新的非堵塞的函数式 Reactive Web 框架，可以用来构建异步的、非堵塞的、事件驱动的服务，在伸缩性方面表现非常好。


熔断策略

## *注册中心*
1. **Consul**

Consul 的优势：

	1. 使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 
	2. 采用的是 Paxos, 而 etcd 使用的则是 Raft。
	3. 支持多数据中心，内外网的服务采用不同的端口进行监听。
	4. 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等。 zookeeper 和 etcd 均不提供多数据中心功能的支持。
	5. 支持健康检查。 etcd 不提供此功能。
	6. 支持 http 和 dns 协议接口。 zookeeper 的集成较为复杂, etcd 只支持 http 协议。
	7. 官方提供 web 管理界面, etcd 无此功能。
	
- 服务注册/发现

	Consul提供的服务注册/发现功能在数据强一致性和分区容错性上都有非常好的保证，但在集群可用性下就会稍微差一些
	
- 数据强一致性保证

	Consul采用了一致性算法Raft来保证服务列表数据在数据中心中各Server下的强一致性，这样能保证同一个数据中心下不
管某一台Server Down了，请求从其他Server中同样也能获取的最新的服务列表数据。数据强一致性带来的副作用是当数据
在同步或者Server在选举Leader过程中，会出现集群不可用。
- 多数据中心

	Consul支持多数据中心(Data Center),多个数据中心之间通过Gossip协议进行数据同步。多数据中心的好处是当某个数据
中心出现故障时，其他数据中心可以继续提供服务，提升了可用性。
- 健康检查

	Consul支持基本硬件资源方面的检查，如：CPU、内存、硬盘等
- Key/Value存储

	Consul支持Key/Value存储功能，可以将Consul作为配置中心使用，可以将一些公共配置信息配置到Consul，然后通过Consul
提供的 HTTP API来获取对应Key的Value。

	- Agent
1. 是一个守护线程
2. 跟随Consul应用启动而启动
3. 负责检查、维护节点同步

	- Client
1. 转发所有请求给Server
2. 无状态，不持久化数据
3. 参与LAN Gossip的健康检查

	- Server
1. 持久化数据
2. 转发请求给Server-Leader
3. 参与Server-Leader选举
4. 通过WAN Gossip，与其他数据中心交换数据

	- Server-Leader
1. 响应RPC请求
2. 服务列表数据同步给Serve

2. **Eureka**

C(一致性)、A(可用性)和P(分区容错性)Zookeeper保证的是CP, 而Eureka则是AP。Zookeeper保证CP

**Eureka的原理？**

- 服务提供方启动后将注册到 注册中心，提供IP, 名字，什么服务等信息，
- 服务调用方作为客户端注册到注册中心后，拉取注册中心的服务列表，在通过负载均衡调用对应
的服务提供方。
- 注册中心可以建立集群，生成多台eureka，注册中心为了监测各个服务的心跳，将在每30S - 向所注册的服务发起请求判断
- 服务是否挂掉，如果挂掉90S后将会将服务从注册中心剔除。
- 一个服务可以监测多台服务实例，从而可实现均衡负载

	
**Eureka是怎么保证AP的？**	

Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个
Eureka注册或时如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的
信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么
Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：
- Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
- Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

**Eureka是怎样进行服务剔除的？**	
- 关闭了自我保护
- 如果开启了自我保护，需要进一步判断是 Eureka Server 出了问题，还是 Eureka Client 出了问题，如果是 Eureka Client 出了问题则进行剔除。
- 怎样判断是Server还是Clinet出问题了？
如果大量服务都续约失败，则认为是自己出问题了（如自己断网了），也就不剔除了；反之，则是 Eureka Client的问题，需要进行剔除。
而自我保护阈值是区分Eureka Client 还是 Eureka Server 出问题的临界值：如果超出阈值就表示大量服务可用，少量服务不可用，则判
定是 Eureka Client 出了问题。如果未超出阈值就表示大量服务不可用，则判定是 Eureka Server 出了问题。
自我保护阈值 = 服务总数 * 每分钟续约数 * 自我保护阈值因子。
每分钟续约数 =（60S/ 客户端续约间隔）
最后自我保护阈值的计算公式为：
自我保护阈值 = 服务总数 * （60S/ 客户端续约间隔） * 自我保护阈值因子。
- 剔除服务的步骤：
	- 删除服务信息，从 registry 中删除服务。
	- 更新队列，将当前剔除事件保存到更新队列中。
	- 清空二级缓存，保证数据的一致性。

参考：
	[Eureka的架构与原理](https://www.infoq.cn/article/jlDJQ*3wtN2PcqTDyokh)

**Zookeeper**
Leader-Server：Leader负责进行投票的发起和决议，更新系统中的数据状态
Server：Server中存在两种类型：Follower和Observer。其中Follower接受客户端的请求并返回结果(事务请求将转发给Leader处理)，并在选举过程中参与投票；Observer与Follower功能一致，但是不参与投票过程，它的存在是为了提高系统的读取速度
Client：请求发起方，Server和Client之间可以通过长连接的方式进行交互。如发起注册或者请求集群信息等。
最后我们通过一张表格大致了解Eureka、Consul、Zookeeper的异同点。选择什么类型的服务注册与发现组件可以根据自身项目要求决定。

组件名|语言|CAP|一致性算法|服务健康检查|对外暴露接口|Spring Cloud集成
--|:--|:--|:--|:--|:--|:--|:
Eureka|Java|AP|无|可配支持|HTTP|已集成
Consul|Go|CP|Raft|支持|HTTP/DNS|已集成
Zookeeper|Java|CP|Paxos|支持|客户端|已集成

[服务发现的比较](https://www.jianshu.com/p/e72e3b208a0b)

## *Spring Cloud Gateway*



# **综述**
web服务器选型：

feign ribbon zuul Hystrix

redis缓存

Eureka：服务注册和发现的产品
	服务中心又称注册中心，管理各种服务功能包括服务的注册、发现、熔断、负载、降级等
	Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers.

mvn clean package
# 分别以peer1和peeer2 配置信息启动eureka
java -jar spring-cloud-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer1
java -jar spring-cloud-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer2

java -jar spring-cloud-consumer-0.0.1-SNAPSHOT.jar

java -jar spring-cloud-eureka-0.0.1-SNAPSHOT.jar

java -jar spring-cloud-producer-0.0.1-SNAPSHOT.jar

发布平台
jenkins

## 网关

Nginx 静态编译?
Nginx 在启动后，会有一个 Master 进程和多个 Worker 进程，Master 进程和 Worker 进程之间是通过进程间通信进行交互的
Master和worker线程的作用是啥？
Nginx的工作原理？

Zuul ：


Feth


hystrix:
  command:
    default:
      execution:
        timeout:
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 600000

ribbon:
  ConnectTimeout: 30000
  ReadTimeout: 600000