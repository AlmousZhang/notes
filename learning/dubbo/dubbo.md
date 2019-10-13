Invocation:
它有2种类型的Invoker
1.本地执行类的Invoker
server端：比如有一个dubbo接口demoService.sayHello，在本项目中执行 demoService.sayHello，就通过InjvmExporter来进行反射执行demoService.sayHello就可以了。

2.远程通信类的Invoker
client端：要执行 demoService.sayHello，它封装了DubboInvoker进行远程通信，发送要执行的接口给server端。
server端：采用了AbstractProxyInvoker执行了DemoServiceImpl.sayHello,然后将执行结果返回发送给client.


Directory
	Directory就是装载invoker的文件目录
	两个重要Directory
	StaticDirectory：静态目录服务，他的Invoker是固定的。（多注册中心）
	RegistryDirectory：注册目录服务，他的Invoker集合数据来源于zk注册中心的，他实现了NotifyListener接口，这个接口中的notify方法就是注册中心的回调，也就是它之所以能根据注册中心动态变化的根源所在.。
整个过程有一个重要的map变量，methodInvokerMap（它是数据的来源；同时也是notify的重要操作对象，重点是写操作。）

group 多分支集合，方法methodInvokerMap 以方法为key

Router
利用Router，可以从多个服务提者方中选择一个进行调用
分类
主要是3个实现类：
ConditionRouter（条件路由）：条件路由主要就是根据dubbo管理控制台配置的路由规则来过滤相关的invoker
MockInvokersSelector：主要根据参数，判断是否需要筛选出正常的（非mock的）invoker 或者 mock的invoker
ScriptRouter（脚本路由）：

Router在应用隔离,读写分离,灰度发布都有使用

LoadBalance
与Router功能类似，利用负载均衡策略（random,roundrobin,leastactive），从多个服务提者方中选择一个进行调用

集中负载均衡的算法的实现。


Protocol 是服务域，它是 Invoker 暴露和引用的主功能入口，它负责 Invoker 的生命周期管理。
消费方调用过程中，dubbo究竟做了什么？
a、在Directory中找出本次集群中的全部invokers
b、在Router中,将上一步的全部invokers挑选出满足条件的invokers
c、在LoadBalance中,将上一步的能正常的执行invokers中,根据配置的负载均衡策略,挑选出需要执行的invoker


集群容错设计
	
1.服务发布中做了哪些事情？
2.dubbo都有哪些协议，他们之间有什么缺点？缺省值是什么
3.什么是本地暴露和远程暴露？


服务提供者能实现失效踢出是根据什么原理?
dubbo中zookeeper做注册中心,如果注册中心集群都挂掉,那发布者和订阅者还能通信吗?

zookeeper的本地缓存

服务发布过程：
1.暴露本地服务
2.暴露远程服务
3.启动netty
4.连接zookerper
5.到zookerper注册
6.监听


Adaptive类

服务引用：
	dubbo服务应用的过程，原理
	既然你提到了dubbo的服务引用中封装通信细节是用到了动态代理,那请问创建动态代理常用的方式有哪些,他们又有什么区别?dubbo中用的是哪一种?(高频题)
	除了JDK动态代理和CGLIB动态代理外,还知不知道其他实现代理的方式?(区分度高)

原理：
装饰者设计模式->静态代理->JDK、cglib、Javassist优缺点对比->AOP源码


RPC框架
gRPC
Dubbo
SpringCloud

RPC 的核心功能主要由 5 个模块组成:
	客户端（Client）：服务调用方。
	客户端存根（Client Stub）：存放服务端地址信息，将客户端的请求参数数据信息打包成网络消息，再通过网络传输发送给服务端。
	服务端存根（Server Stub）：接收客户端发送过来的请求消息并进行解包，然后再调用本地服务进行处理。
	服务端（Server）：服务的真正提供者。
	Network Service：底层传输，可以是 TCP 或 HTTP

RPC技术难点：
	服务寻址：服务注册中心
	数据流序列化和反序列化：
	网络传输：
	
