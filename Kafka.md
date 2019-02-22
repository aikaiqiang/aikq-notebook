KafKa
[博客教程-1](https://blog.csdn.net/zhouyou1986/article/details/42319461)
[博客教程-2](https://www.cnblogs.com/huxi2b/tag/Kafka/)
###  kafka简介
#### kafka起源
Kafka是由LinkedIn开发并开源的分布式消息系统，2012年捐赠给Apache基金会，采用Scala语言，运行在JVM中，最新版本2.0.1[下载地址](http://kafka.apache.org/downloads)

#### kafka设计目标
Kafka是一种分布式的，基于发布/订阅的消息系统
设计目标：
- 以时间复杂度O(1)的方式提供消息持久化能力，对TB级别的数据也能保证常数时间复杂度的访问性能；
- 高吞吐率。在低配机器上也能保证每秒10万条以上消息的传输；
- 支持kafka server间的消息分区，分布式消费，同时保证每个Partition内消息的顺序传输；
- 支持离线数据和实时数据处理
- scale out，支持在线水平扩展

#### 使用消息系统的好处
解耦，冗余，扩展性，灵活性&峰值处理能力，可恢复性，顺序保证，缓冲，异步通信

#### 对比常用消息中间件
||ActiveMQ|RabbitMQ|Kafka|
|------|------|------|------|
|produce容错，是否丢失数据||有ack模型，也有事务模型，保证至少不会丢失数据。ack模型可能会有重复消息，事务模型保证完全一致|批量形式下可能会丢失数据；非批量形式下：1.使用同步模式可能会有重复数据，2.使用异步模式可能会丢失数据|
|consumer容错，是否丢失数据||有ack模型，数据不会丢失，但可能会有重复数据|批量形式下可能会丢数据。非批量形式下，可能会重复处理数据（ZK写offset是异步的）|
|架构模型|基于JMS协议|基于AMQP模型，比较成熟，但更新超慢。RabbitMQ的broker由Exchange，Binding，queue组成，其中exchange和binding组成了消息的路由键；客户端Producer通过连接channel和server进行通信，Consumer从queue获取消息进行消费（长连接，queue有消息就会推送到consumer端，consumer循环从输入流读取数据）；RabbitMQ以broker为中心；有消息确认机制|producer，broker，consumer，以consumer为中心，消息的消费信息保存在客户端consumer上，consumer根据消费点从broker上批量pull数据；无消息确认机制|
|吞吐量||RabbitMQ在吞吐量方面稍逊于Kafka，两者出发点不一样，RabbitMQ支持消息的可靠传递，支持事务，不支持批量操作；基于存储的可靠性的要求存储可以采用内存或者硬盘|Kafka具有高吞吐量，内部采用消息的批量处理，zero-copy机制，数据的存储和获取是本地磁盘顺序批量操作，具有O(1)的复杂度，消息处理的效率很高|
|可用性||RabbitMQ支持miror的queue，主queue失效，miror queue接管|Kafka的broker支持主备模式|
|集群负载均衡||RabbitMQ的负载均衡需要单独的loadbalancer进行支持|Kafka采用zookeeper对集群中broker，consumer进行管理，可以注册topic到zookeeper上；通过zookeeper的协调机制，producer保存对应topic的broker信息，可以随机或者轮询发送到broker上，并且producer可以基于语义指定分片，消息发送到broker的某分片上|


### Kafka架构
#### Kafka术语
- Topic
用于划分Message的逻辑概念，一个Topic可以分布在多个Broker上。
- Partition
是Kafka中横向扩展和一切并行化的基础，是物理上的概念，每个Topic都至少被切分为1个Partition
- Offset
消息在Partition中的编号，编号顺序不跨Partition
- Consumer
用于从Broker中取出/消费Message
- Consumer Group
每个Consumer属于一个特定的Consumer Group（可以为每个Consumer指定group name则属于默认的group）
- Producer
用户往Broker中发送/上产消息Message
- Replication
Kafka支持以Partition为单位对Message进行冗余备份，每个Partition都可以配置至少1个Replication(当仅1个Replication时即仅该Partition本身)
- Leader
每个Replication集合中的Partition都会选出一个唯一的Leader，所有的读写请求都由Leader处理。其他Replicas从Leader处把数据更新同步到本地，过程类似大家熟悉的MySQL中的Binlog同步
- Broker
Kafka集群包含一个或多个服务器，这种服务器被称为broker。Kafka中使用Broker来接受Producer和Consumer的请求，并把Message持久化到本地磁盘。每个Cluster当中会选举出一个Broker来担任Controller，负责处理Partition的Leader选举，协调Partition迁移等工作
- ISR（In-Sync Replica）
是Replicas的一个子集，表示目前Alive且与Leader能够“Catch-up”的Replicas集合。由于读写都是首先落到Leader上，所以一般来说通过同步机制从Leader上拉取数据的Replica都会和Leader有一些延迟(包括了延迟时间和延迟条数两个维度)，任意一个超过阈值都会把该Replica踢出ISR。每个Partition都有它自己独立的ISR



###  kafka单机下载安装
#### 下载
kafka 下载地址：
http://mirrors.hust.edu.cn/apache/kafka/2.0.1/kafka_2.12-2.0.1.tgz
```
# wget工具下载
wget http://mirrors.hust.edu.cn/apache/kafka/2.0.1/kafka_2.12-2.0.1.tgz
```
#### 解压
```
tar zxf kafka_2.12-2.0.1.tgz -C /aikq/kafka
```

#### 修改配置文件config/server.properties
配置文件解析，[参考地址](https://blog.csdn.net/lizhitao/article/details/25667831)
```
cd /aikq/kafka/kafka_2.12-2.0.1
# 修改配置文件config/server.properties
vim config/server.properties

# 后台启动kafka
./kafka-server-start.sh ../config/server.properties &

# 新建xshell连接-1，生成者
bin/kafka-console-producer.sh --broker-list ip(服务器ip):9092 --topic test
# 新建xshell连接-2，消费者
bin/kafka-console-consumer.sh --bootstrap-server ip:9092 --topic test --from-beginning
bin/kafka-console-consumer.sh --zookeeper ip:2181 --topic test --from-beginning（老版本消费命令）

# 生成者产生消息，消费者可以消费消息

```

###  kafka伪集群模式
broker-0：
vim  config/server-0.properties
```
broker.id=0
listeners=PLAINTEXT://:9092
port=9092
#host.name=192.168.1.177
num.network.threads=4
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/tmp/kafka-logs
num.partitions=5
num.recovery.threads.per.data.dir=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
log.cleaner.enable=false
zookeeper.connect=localhost:2181
zookeeper.connection.timeout.ms=6000
queued.max.requests =500
log.cleanup.policy = delete
```
broker-1：
vim  config/server-1.properties
```
broker.id=1
listeners=PLAINTEXT://:9093
port=9093
#host.name=192.168.1.177
num.network.threads=4
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/tmp/kafka-logs
num.partitions=5
num.recovery.threads.per.data.dir=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
log.cleaner.enable=false
zookeeper.connect=localhost:2181
zookeeper.connection.timeout.ms=6000
queued.max.requests =500
log.cleanup.policy = delete
```
broker-2：
vim  config/server-2.properties
```
broker.id=2
listeners=PLAINTEXT://:9094
port=9094
#host.name=192.168.1.177
num.network.threads=4
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/tmp/kafka-logs
num.partitions=5
num.recovery.threads.per.data.dir=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
log.cleaner.enable=false
zookeeper.connect=localhost:2181
zookeeper.connection.timeout.ms=6000
queued.max.requests =500
log.cleanup.policy = delete
```

分别启动这个三个broker
```
bin/kafka-server-start.sh config/server-0.properties &   #启动broker-0
bin/kafka-server-start.sh config/server-1.properties &   #启动broker-1
bin/kafka-server-start.sh config/server-2.properties &   #启动broker-2
```

生产者-消费者集群模式
```
bin/kafka-console-producer.sh --topic topic_1 --broker-list 192.168.1.177:9092,192.168.1.177:9093,192.168.1.177:9094
```



###  kafka集群模式