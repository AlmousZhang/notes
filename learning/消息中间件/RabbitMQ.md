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

## 原理


