## *Spring Boot 2.0*
	Webflux 是一个全新的非堵塞的函数式 Reactive Web 框架，可以用来构建异步的、非堵塞的、事件驱动的服务，在伸缩性方面表现非常好。


熔断策略

## *注册中心*
Consul 的优势：
	1 使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 
	2 采用的是 Paxos, 而 etcd 使用的则是 Raft。
	3 支持多数据中心，内外网的服务采用不同的端口进行监听。
	4 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等。 zookeeper 和 etcd 均不提供多数据中心功能的支持。
	5 支持健康检查。 etcd 不提供此功能。
	6 支持 http 和 dns 协议接口。 zookeeper 的集成较为复杂, etcd 只支持 http 协议。
	7 官方提供 web 管理界面, etcd 无此功能。
综合比较, Consul 作为服务注册和配置管理的新星, 比较值得关注和研究

## *Spring Cloud Gateway*



# **总数**
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