## 面试题
1. Apache Kafka 是什么?
	基于发布与订阅的消息系统，Kafka是一个分布式的，可分区的，冗余备份的持久日志服务。主要用于处理活跃的流式数据。
	降低系统组网复杂度。
	降低编程复杂度，各个子系统不在是相互协商接口，各个子系统类似插口插在插座上，Kafka 承担高速数据总线的作用。

2. Kafka的特点？
- 同时为发布和订阅提提供高吞吐量，
- 可进行持久化操作。将消息持久化到磁盘，可用于批量消费及实时应用程序。通过将数据持久化到硬盘，以及replication ，可以防止数据丢失。
- 分布式系统，易于向外扩展。所有的 Producer、Broker 和Consumer 都会有多个，均为分布式的。并且，无需停机即可扩展机器。
- 消息被处理的状态是在 Consumer 端维护，而不是由 Broker 端维护。当失败时，能自动平衡。
	- 消息是否被处理完成，是通过 Consumer 提交消费进度给 Broker ，而不是 Broker 消息被 Consumer 拉取后，就标记为已消费。
	- 当 Consumer 异常崩溃时，可以重新分配消息分区到其它的 Consumer 们，然后继续消费
- 支持 online 和 offline 的场景。

3. Kafka的设计要点
- 吞吐量
	- 数据磁盘持久化：消息不在内存中 Cache ，直接写入到磁盘，充分利用磁盘的顺序读写性能。
	- zero-copy：减少 IO 操作步骤
	- 数据批量发送
	- 数据压缩
	- Topic 划分为多个 Partition ，提高并行度。
	```
	Kafka 以 Topic 来进行消息管理，每个 Topic 包含多个 Partition ，每个 Partition 对应一个逻辑 log ，有多个 segment 文件组成。
	每个 segment 中存储多条消息，消息 id 由其逻辑位置决定，即从消息 id 可直接定位到消息的存储位置，避免 id 到位置的额外映射。
	每个 Partition 在内存中对应一个 index ，记录每个 segment 中的第一条消息偏移。
	综述：发布者发到某个 Topic 的消息会被均匀的分布到多个 Partition 上（随机或根据用户指定的回调函数进行分布），Broker 收到发布消息往对应 Partition 的最后一个 segment 上添加该消息。
	当某个 segment上 的消息条数达到配置值或消息发布时间超过阈值时，segment上 的消息会被 flush 到磁盘，只有 flush 到磁盘上的消息订阅者才能订阅到，segment 达到一定的大小后将不会再往该 segment 写数据，Broker 会创建新的 segment 文件。
	```
- 负载均衡
	- Producer 根据用户指定的算法，将消息发送到指定的 Partition 中。
	- Topic 存在多个 Partition ，每个 Partition 有自己的replica ，每个 replica 分布在不同的 Broker 节点上。多个Partition 需要选取出 Leader partition ，Leader Partition 负责读写，并由 Zookeeper 负责 fail over 。
	- 相同 Topic 的多个 Partition 会分配给不同的 Consumer 进行拉取消息，进行消费。
- 拉取系统
由于 Kafka Broker 会持久化数据，Broker 没有内存压力，因此， Consumer 非常适合采取 pull 的方式消费数据，具有以下几点好处：
	- 简化 Kafka 设计
	- Consumer 根据消费能力自主控制消息拉取速度。 
	- Consumer 根据自身情况自主选择消费模式，例如批量，重复消费，从尾端开始消费等。
- 可扩展性
	- 当需要增加 Broker 节点时，新增的 Broker 会向 Zookeeper 注册，而 Producer 及 Consumer 会根据注册在 Zookeeper 上的 watcher 感知这些变化，并及时作出调整。
	- 当新增和删除 Consumer 节点时，相同 Topic 的多个 Partition 会分配给剩余的 Consumer 们。
参考：[为什么Kafka这么快](https://mp.weixin.qq.com/s/pzVS7r3QaQPFwob-fY8b4A)

4. Kafka 为什么要将 Topic 进行分区？
- Topic 只是逻辑概念，面向的是 Producer 和 Consumer ，而 Partition 则是物理概念。如果 Topic 不进行分区，而将 Topic 内的消息存储于一个 Broker，那么关于该 Topic 的所有读写请求都将由这一个 Broker 处理，吞吐量很容易陷入瓶颈，这显然是不符合高吞吐量应用场景的。
- 有了 Partition 概念以后，假设一个 Topic 被分为 10 个 Partitions ，Kafka 会根据一定的算法将 10 个 Partition 尽可能均匀的分布到不同的 Broker（服务器）上。
- 当 Producer 发布消息时，Producer 客户端可以采用 random、key-hash 及轮询等算法选定目标 Partition ，若不指定，Kafka 也将根据一定算法将其置于某一分区上。
- 当 Consumer 拉取消息时，Consumer 客户端可以采用 Range、轮询 等算法分配 Partition ，从而从不同的 Broker 拉取对应的 Partition 的 leader 分区。

5. Kafka 的应用场景有哪些？
- 消息队列：比起大多数的消息系统来说，Kafka 有更好的吞吐量，内置的分区，冗余及容错性，这让 Kafka 成为了一个很好的大规模消息处理应用的解决方案。消息系统一般吞吐量相对较低，但是需要更小的端到端延时，并常常依赖于 Kafka 提供的强大的持久性保障。在这个领域，Kafka 足以媲美传统消息系统，如 ActiveMQ 或 RabbitMQ
- 行为跟踪：Kafka 的另一个应用场景，是跟踪用户浏览页面、搜索及其他行为，以发布订阅的模式实时记录到对应的 Topic 里。那么这些结果被订阅者拿到后，就可以做进一步的实时处理，或实时监控，或放到 Hadoop / 离线数据仓库里处理。
- 元信息监控:作为操作记录的监控模块来使用，即汇集记录一些操作信息，可以理解为运维性质的数据监控吧。
- 日志收集:Kafka 忽略掉文件的细节，将其更清晰地抽象成一个个日志或事件的消息流。这就让 Kafka 处理过程延迟更低，更容易支持多数据源和分布式数据处理。比起以日志为中心的系统比如 Scribe 或者 Flume 来说，Kafka 提供同样高效的性能和因为复制导致的更高的耐用性保证，以及更低的端到端延迟。
- 流处理
- 事件源


##几个重要的概念##
- Topic：特指 Kafka 处理的消息源（feeds of messages）的不同分类。
- Partition：Topic 物理上的分组（分区），一个 Topic 可以分为多个 Partition 。每个 Partition 都是一个有序的队列。Partition 中的每条消息都会被分配一个有序的 id（offset）。
	- replicas：Partition 的副本集，保障 Partition 的高可用。
	- leader：replicas 中的一个角色，Producer 和 Consumer 只跟 Leader 交互。
	- follower：replicas 中的一个角色，从 leader 中复制数据，作为副本，一旦 leader 挂掉，会从它的 followers 中选举出一个新的 leader 继续提供服务。

- Message：消息，是通信的基本单位，每个 Producer 可以向一个Topic（主题）发布一些消息。
- Producers：消息和数据生产者，向 Kafka 的一个 Topic 发布消息的过程，叫做 producers
- Consumers：消息和数据消费者，订阅 Topic ，并处理其发布的消息的过程，叫做 consumers
```Consumer group：每个 Consumer 都属于一个 Consumer group，每条消息只能被 Consumer group 中的一个 Consumer 消费，但可以被多个 Consumer group 消费。```
- Broker：缓存代理，Kafka 集群中的一台或多台服务器统称为 broker
```Controller：Kafka 集群中，通过 Zookeeper 选举某个 Broker 作为 Controller ，用来进行 leader election 以及 各种 failover```
- ZooKeeper：Kafka 通过 ZooKeeper 来存储集群的 Topic、Partition 等元信息等

