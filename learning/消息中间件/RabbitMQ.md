## 安装

默认账号密码是：是guest/guest

1. **拉取镜像**
```
docker pull rabbitmq:3.6.15-management
```
2. **启动单节点RabbitMQ**
```
docker run -d --hostname localhost --name myrabbit -p 15672:15672 -p 5672:5672 rabbitmq:3.6.15-management
```
参数说明：
-d 后台进程运行
hostname RabbitMQ主机名称
name 容器名称
-p port:port 本地端口:容器端口
-p 15672:15672 http访问端口
-p 5672:5672 amqp访问端口

3. **搭建RabbitMQ集群**
```
docker run -d --hostname rabbit1 --name myrabbit1 -p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3.6.15-management

docker run -d --hostname rabbit2 --name myrabbit2 -p 5673:5672 --link myrabbit1:rabbit1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3.6.15-management

docker run -d --hostname rabbit3 --name myrabbit3 -p 5674:5672 --link myrabbit1:rabbit1 --link myrabbit2:rabbit2 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3.6.15-management
```
注意点：
1. 多个容器之间使用“--link”连接，此属性不能少；
2. Erlang Cookie值必须相同，也就是RABBITMQ_ERLANG_COOKIE参数的值必须相同，原因见下文“配置相同Erlang Cookie”部分；
步骤二：加入RabbitMQ节点到集群

设置节点1：
```
docker exec -it myrabbit1 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
exit
```
设置节点2，加入到集群：
```
docker exec -it myrabbit2 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbit1
rabbitmqctl start_app
exit
```
参数“--ram”表示设置为内存节点，忽略次参数默认为磁盘节点。

设置节点3，加入到集群：
```
docker exec -it myrabbit3 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbit1
rabbitmqctl start_app
exit
```


[参考](https://www.cnblogs.com/vipstone/p/9362388.html)

4. **docker-compose 安装搭建 RabbitMQ 集群**

todo
[待验证](https://michael728.github.io/2019/06/07/docker-rabbitmq-env/)

## 使用
- AMQP协议，是一种二进制协议;实现应用程序的异步和解耦，同时也能起到消息缓冲，消息分发的作用

**重要概念**
- 虚拟主机：一个虚拟主机持有一组交换机、队列和绑定。为什么需要多个虚拟主机呢？很简单，RabbitMQ当中，_用户只能在虚拟主机的粒度进行权限控制。_因此，如果需要禁止A组访问B组的交换机/队列/绑定，必须为A和B分别创建一个虚拟主机。每一个RabbitMQ服务器都有一个默认的虚拟主机“/”。
- 交换机：Exchange 用于转发消息，但是它不会做存储_，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。exchange是一个消息的agent，每一个虚拟的host中都有定义。它的职责是把message路由到不同的queue中。producer deliver消息的时候会把routing-key add到 message header中。routing-key只是一个messgae的attribute。
- 队列：消息到交换机的时候，交互机会转发到对应的队列中，那么究竟转发到哪个队列，就要根据该路由键
- 绑定：交换机需要和队列相绑定

**Directed Exchange**

路由键exchange，该交换机收到消息后会把消息发送到指定routing-key的queue中。那消息交换机是怎么知道的呢？其实，producer deliver消息的时候会把routing-key add到 message header中。routing-key只是一个messgae的attribute。

**Default Exchange**

这种是特殊的Direct Exchange，是rabbitmq内部默认的一个交换机。该交换机的name是空字符串，所有queue都默认binding 到该交换机上。所有binding到该交换机上的queue，routing-key都和queue的name一样。

**Topic Exchange**

通配符交换机，exchange会把消息发送到一个或者多个满足通配符规则的routing-key的queue。其中_表号匹配一个word，#匹配多个word和路径，路径之间通过.隔开。如满足a._.c的routing-key有a.hello.c；满足#.hello的routing-key有a.b.c.helo。

**Fanout Exchange**

扇形交换机，该交换机会把消息发送到所有binding到该交换机上的queue。这种是publisher/subcribe模式。用来做广播最好。
所有该exchagne上指定的routing-key都会被ignore掉。

**Header Exchange**

设置header attribute参数类型的交换机。

## 运维
- 进入docker部署的RabbitMQ: `docker exec -it myrabbit1 bash`
- 查看集群信息：`rabbitmqctl cluster_status`
- 用户管理
`rabbitmqctl add_user dahe dahe123456`
rabbitmqctl set_permissions -p / dahe ".*" ".*" ".*"
rabbitmqctl set_user_tags dahe administrator


## 原理
AMQP协议：
生产者将消息发送给交换器，交换器和队列绑定。当生产者发送消息时携带的RouthingKey与绑定的BingingKey相匹配时，消息被存入相应的队列中。消费者可以订阅相应的队列来获取消息

生产者(Producer)
消费者(Customer)
队列(Queue)
交换器(Exchange)
路由键(RouthingKey)
绑定(Binding)
连接(Connection)
信道(Chennel)
交换器类型：
fanout
direct
topic
headers

推模式与拉模式

mandatory：当mandatory 参数设为true 时，交换器无法根据自身的类型和路由键找到一个符合条件
的队列，那么RabbitMQ 会调用Basic.Return 命令将消息返回给生产者。当mandatory 参数设置为
false 时，出现上述情形，则消息直接被丢弃。

immediate：当imrnediate 参数设为true 时，如果交换器在将消息路由到队列时发现队列上并不存在
任何消费者，那么这条消息将不会存入队列中。当与路由键匹配的所有队列都没有消费者时，该消息
会通过Basic.Return 返回至生产者(RabbitMQ 3 .0 版本开始去掉了对imrnediate 参数的支持)

概括来说， ma口datory 参数告诉服务器至少将该消息路由到一个队列中， 否则将消息返
回给生产者。imrnediate 参数告诉服务器， 如果该消息关联的队列上有消费者， 则立刻投递:
如果所有匹配的队列上都没有消费者，则直接将消息返还给生产者， 不用将消息存入队列而等
待消费者了。

- 备份交换器：Altemate Exchange

通过参数alternate-exchange设置

- 过期时间(TTL，Time to Live)

消息的TTL:
第一种方法是通过队列属性设置，队列中所有消息都有相同的过期时间。
第二种方法是对消息本身进行单独设置，每条消息的TTL 可以不同

- 死信队列

全称为为Dead-Letter-Exchange ，可以称之为死信交换器，也有人称之为死信邮箱。当
消息在一个队列中变成死信(dead message) 之后，它能被重新被发送到另一个交换器中，这个
交换器就是DLX ，绑定DLX 的队列就称之为死信队列。

- 延迟队列

延迟队列存储的对象是对应的延迟消息，所谓"延迟消息"是指当消息被发送以后，并不
想让消费者立刻拿到消息，而是等待特定时间后，消费者才能拿到这个消息进行消费。
在AMQP 协议中，或者RabbitMQ 本身没有直接支持延迟队列的功能，但是可以通过前面
所介绍的DLX 和TTL 模拟出延迟队列的功能。

- 优先级队列

具有高优先级的队列具有高的优先权，优先级高的消息具备优先被消费的特权。可以通过
设置队列的x - max - priority 参数来实现

- RPC实现

客户端发送请求消息，服务端回复响应的消息。为了接收响应的消息，我们需要在请求消
息中发送一个回调队列。
~ replyTo: 通常用来设置一个回调队列。
~ correlationId : 用来关联请求( request) 和其调用RPC 之后的回复(response ) 。
之后在回调队列接收到回复的消息时，可以根据这个属性匹配到相应的请求。如果回调队
列接收到一条未知correlationld 的回复消息，可以简单地将其丢弃。详细的例子(???)

- 持久化
交换器的持久化、队列的持久化和消息的持久化。
设置了队列和消息的持久化，当RabbitMQ 服务重启之后，消息依旧存在。单单只设置队
列持久化，重启之后消息会丢失;单单只设置消息的持久化，重启之后队列消失，继而消息也
丢

- 生产者确认

通过事务机制实现:
令通过发送方确认(publisher confirm) 机制实现:批量或者一部confirm

- 消费端要点介绍

	- 消息分发
	- 消息顺序性
	- 弃用QueueingConsumer
	
- 消息传输保障

	- At most once: 最多一次。消息可能会丢失，但绝不会重复传输。
	- At least once: 最少一次。消息绝不会丢失，但可能会重复传输。
	如何保证：事务；消息生产者需要配合使用mandatory 参数或者备份交换器来确保消息能够从交换器
路由到队列中，进而能够保存下来而不会被丢弃；消息和队列都需要进行持久化处理，以确保RabbitMQ 服务器在遇到异常情况时不会造成消息丢失；消费者在消费消息的同时需要将autoAck设置为false，然
后通过手动确认的方式去确认己经正确消费的消息，以避免在消费端引起不必要的消息丢失。
	- Exactly once: 恰好一次。每条消息肯定会被传输一次且仅传输一次。
	
用户管理
```
rabbitmqctl list_vhosts
rabbitmqctl add_vhost dahe
rabbitmqctl list_vhosts name tracing
rabbitmqctl trace_on
rabbitmqctl set_permissions -p dahe dahe ".*" ".*" ".*"
rabbitmqctl set_permissions -p dahe dahe "^queue.*" ".*" ".*" ##授予dahe可访问虚拟主机dahe， 在以"queue" 开头的资源上具备可配置权限，并在所有资源上拥有可写、可读的权限
rabbitmqctl clear_permissions -p dahe dahe
```

**存储机制**
	不管是持久化的消息还是非持久化的消息都可以被写入到磁盘。持久化的消息在到达队列
时就被写入到磁盘，并且如果可以，持久化的消息也会在内存中保存一份备份，这样可以提高
一定的性能，当内存吃紧的时候会从内存中清除。非持久化的消息一般只保存在内存中，在内
存吃紧的时候会被换入到磁盘中，以节省内存空间

持久层：队列索引(rabbit_queue_index) 和消息存储(rabbit_msg_store)。 rabbit_queue_index 负责维护队列中落盘消息的信息，包括消息的存储地点、是否己被交付给消费者、是否己被消费者
ack等。
rabbit_msg_store以键值对的形式存储消息，它被所有队列共享，在每个节点中有且只有一个。从技术
层面上来说，rabbit_msg_store 具体还可以分为msg_store_persistent和msg_store_transient, msg_store_persistent 负责持久化消息的持久化，重启后消息不会丢失; msg_store_transient 负责非
持久化消息的持久化，重启后消息会丢失。通常情况下，习惯性地将msg_store_persistent 和msg_store_
transient看成rabbit_msg_store这样一个整体。


