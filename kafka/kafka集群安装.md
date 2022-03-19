##  搭建zookeeper集群

### 主机规划：

192.168.100.7　node1

192.168.100.8    node2

192.168.100.9    node3

### 软件下载地址：

```shell
cd /home/app
wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz
```

###  三台主机hosts文件一致：

```shell
# cat /etc/hosts
192.168.100.7　node1
192.168.100.8  node2
192.168.100.9  node3
```

### 在node1节点上

```shell
[root@node1 app]# tar zookeeper-3.6.2.tar.gz -C /home/app/
[root@node1 app]# mv zookeeper-3.6.2 zookeeper
[root@node1 app]# cd zookeeper/conf/
[root@node1 conf]# cp zoo_sample.cfg zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper
clientPort=2181
server.1=node1:2888:3888
server.2=node2:2888:3888
server.3=node3:2888:3888
```

### 创建dataDir目录创建/tmp/zookeeper

```shell
[root@node1 conf]# mkdir /tmp/zookeeper
[root@node1 conf]# touch /tmp/zookeeper/myid
[root@node1 conf]# echo 1 > /tmp/zookeeper/myid
```

### 将zookeeper文件复制到另外两个节点：

```shell
[root@node1 conf]# scp -r zookeeper-3.6.2/ node2:/home/app/
[root@node1 conf]# scp -r zookeeper-3.6.2/ node3:/home/app/
```

### node2创建dataDir目录创建/tmp/zookeeper

```
[root@node2 conf]# mkdir /tmp/zookeeper
[root@node2 conf]# touch /tmp/zookeeper/myid
[root@node2 conf]# echo 2 > /tmp/zookeeper/myid
```

### node3创建dataDir目录创建/tmp/zookeeper

```shell
[root@node3 conf]# mkdir /tmp/zookeeper
[root@node3 conf]# touch /tmp/zookeeper/myid
[root@node3 conf]# echo 3 > /tmp/zookeeper/myid
```

### 分别在每个节点上启动 zookeeper测试：

```shell
[root@node1 zookeeper]# ./bin/zkServer.sh start
[root@node2 zookeeper]# ./bin/zkServer.sh start
[root@node3 zookeeper]# ./bin/zkServer.sh start
```

### 查看状态

```shell
[root@node1 zookeeper]# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
[root@node2 zookeeper]# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: leader
[root@node3 zookeeper]# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```



## 搭建Kafka集群

### 将Kafka安装包上传到虚拟机，并解压

```shell
[root@node1 ~]# cd /home/app/
[root@node1 app]# wget http://mirrors.hust.edu.cn/apache/kafka/2.8.0/kafka_2.13-2.8.0.tgz
[root@node1 app]# tar -xvzf kafka_2.13-2.8.0.tgz
[root@node1 app]# cd /home/app/kafka_2.13-2.8.0/
```

###  修改server.properties

```shell
[root@node1 app]# cd config
[root@node1 app]# vim server.properties
#指定broker的id
broker.id=0
listeners=PLAINTEXT://node1:9092
advertised.listeners=PLAINTEXT://node1:9092
#指定kafka数据的位置
log.dirs=/home/app/kafka_2.13-2.8.0/data
zookeeper.connect=node1:2181,node2:2181,node3:2181
```

### 将安装好的Kafka的复制到另外两台服务器

```shell
[root@node1 app]# cd /home/app
[root@node1 app]# scp -r kafka_2.13-2.8.0/ node2:$PWD
[root@node1 app]# scp -r kafka_2.13-2.8.0/ node3:$PWD
修改另外两个节点的broker.id分别为1和2
-----node2-----
[root@node2 ~]# cd /home/app/kafka_2.13-2.8.0/config
[root@node2 app]# vim server.properties
#指定broker的id
broker.id=1
listeners=PLAINTEXT://node2:9092
advertised.listeners=PLAINTEXT://node2:9092
-----node3-----
[root@node3 ~]# cd /home/app/kafka_2.13-2.8.0/config
[root@node3 config]# vim server.properties
#指定broker的id
broker.id=2
listeners=PLAINTEXT://node3:9092
advertised.listeners=PLAINTEXT://node3:9092
```

###  配置KAFKA_HOME环境变量

```shell
vim /etc/profile
export KAFKA_HOME=/export/server/kafka_2.13-2.8.0
export PATH=:$PATH:${KAFKA_HOME}
分发到各个节点
scp /etc/profile node2.ip.cn:$PWD
scp /etc/profile node3.ip.cn:$PWD
每个节点加载环境变量
source /etc/profile
```

### 启动服务器

```shell
#启动ZooKeeper
nohup bin/zookeeper-server-start.sh config/zookeeper.properties &
#启动kafka
cd /export/server/kafka_2.12-2.4.1
nohup bin/kafka-server-start.sh config/server.properties &
#测试kafka集群是否启动成功
bin/kafka-topics.sh --bootstrap-server node1.ip.cn:9092 --list
```

### Kafka一键启动/关闭

### 在几点中创建/export/onekey 目录

```shell
cd /home/app/onekey
```

### 准备slave配置文件，用于保存要启动哪几个节点上的kafka

```shell
node1
node2
node3
```

### 编写start-kafka.sh脚本

```shell
vim start-kafka.sh
cat ./slave | while read line
do
{
echo $line
ssh $line "source /etc/profile; export JMX_PORT=9988;
nohup ${KAFKA_HOME}/bin/kafka-server-start.sh ${KAFKA_HOME}/config/server.properties > /dev/nul* 2>&1"&
}&
wait
done
```

### 编写stop-kafka.sh脚本

```shell
vim stop-kafka.sh
cat ./salve | while read line
do
{
echo $line
ssh $line "source /etc/profile; jps |grep Kafka |cut -d ' ' -f1 |xargs kill -9"&
}&
wait
done
```

## 基础操作

### 创建topic（主题）

```shell
#创建名为test的主题
bin/kafka-topics.sh --create --bootstrap-server node1.ip.cn:9092 --topic test
#查看目前kafka中的主题
bin/kafka-topics.sh --list --bootstrap-server node1.ip.cn:9092
```

### 生产消息到kafka

使用kafka内置的测试程序，生产一些消息到kafka的test主题中

```shell
bin/kafka-console-producer.sh --broker-list node1.ip.cn:9092 --topic test
```

### 从kafka消费消息

使用下面的命令来消费test主题中的消息

```shell
bin/kafka-console-consumer.sh --bootstrap-server node1.ip.cn:9092 --topic test --from-beginning
```

### 使用Kafka Tools操作Kafka

### 连接Kafka集群

安装kafka Tools后启动kafka

## Kafka基准测试

### 基于一个分区一个副本的基准测试

#### 创建topic

```shell
bin/kafka-topics.sh --zookeeper node1:2181 --create --topic benchmark --partitions 1 --replication-factor 1
```

#### 生成消息基本测试

在生产环境中，推荐使用生产5000W消息，这样性能数据会更准确些。为了方便测试，课程上演示500W的消息作为基准测试。

```shell
bin/kafka-producer-perf-test.sh --topic benchmark --num-records 5000000 --throughput -1 --record-size 1000 --producer-props bootstrap.servers=node1:9092,node2:9092,node3:9092 acks=1
```

> bin/kafka-producer-perf-test.sh
>
> --topoc topic的名字
>
> --num-records 总共指定生产数据量（默认5000W）
>
> --throughput 指定吞吐量—限流（-1不指定）
>
> --record-size record数据大小（字节）
>
> --producer-props bootstrap.servers=node1:9092,node2:9092,node3:9092 acks=1 指定kafka集群地址，ACK模式；  

测试结果

| 吞吐量       | 93092.533979 records/sec<br>每秒9.3W条记录 |
| ------------ | ------------------------------------------ |
| 吞吐速率     | (88.78/sec)<br/>每秒约89MB数据             |
| 平均延迟时间 | 346.62 ms avg latency                      |
| 最大延迟时间 | 1003.00 ms max latency                     |

#### 消费消息基准测试

```shell
bin/kafka-consumer-perf-test.sh --broker-list node1:9092,node2:9092,node3:9092 --topic benchmark --fetch-size 1048576 --messages 5000000
```

> bin/kafka-consumer-perf-test.sh
>
> --broker-list 指定kafka集群地址
>
> --topic  指定topic的名称
>
> --fetch--size 每次拉取的数据大小
>
> --messages 总共要消费的消息个数

测试结果

| data.consumed.in.MB<br>共计消费的数据    | 4768.3434MB               |
| ---------------------------------------- | ------------------------- |
| MB.sec<br/>每秒消费的数量                | 445.6006<br/>每秒445MB    |
| data.consumed.in.nMsg<br/>共计消费的数量 | 5000000                   |
| nMsg.sec<br/>每秒的数量                  | 467234.0518<br/>每秒46.7W |

## Kafka中得重要概念

- broker
  - kafka服务器，生产者、消费者都要连接broker
  - 一个集群由多个broker组成，功能实现kafka集群得负载均衡、容错

-  producer：生产者
- consumer：消费者
- topic：主题，一个Kafka集群中，可以包含多个topic。一个topic可以包含多个分区
  - 是一个逻辑结构，生产、消费消息都需要指定topic
- partiton：kafka集群得分布式就是由分区来实现的。一个topic中得消息可以分布在topic中得不同partition中
- replica：副本，实现kafka集群的容错，实现partition的容错。一个topic至少应该包含大于一个的副本
- consumer group：消费者组，一个消费者组中的消费者可以共同消费topic中的分区数据。每一个消费者组都是一个唯一的名字。配置group.id一样的消费者是属于同一个组中
- offset：偏移量。相对消费者、partition来说。可以通过offset来拉取数据

### 消费者组

一个消费者组中可以包含多个消费者，共同来消费topic中的数据

一个topic中如果只有一个分区，那么这个分区智能被某个组中的一个消费者消费

有多少分区，那么就可以被同一个组内的多少个消费者消费

设置test topic为2个分区

```shell
bin/kafka-topic.sh --zookeeper node1:2181 -alter --partitons 2 --topic test
```

### 幂等性

- 生产者消息重复的问题
  - Kafka生产消息到partition，如果直接发送消息，Kafka会将消息保存到分区中，但Kafka会返回一个ack给生产者，标识当前操作是否成功，是否已经保存了这条消息。如果ack响应的过程失败了，此时生产者会重试，继续发送没有发送成功的消息，Kafka又会保存一条一摸一样的消息

- 在Kafka中可以开启幂等性
  - 当kafka的生产消息时，会增加一个pid（生产者的唯一编号）和sequence number（针对消息的一个递增序列）
  - 发送消息，会连着pid和sequence number一起放松
  - Kafka接收到消息，会将消息和pid、sequence number一并保存下来
  - 如果ack响应失败，生产者重试，再次发送消息时，Kafka会根据pid、sequence number是否需要再保存一条消息
  - 判断条件：生产者发送过来的sequence number是否小于等于parition中消息对应的sequence

### kafka中的副本机制

#### 生产者的分区写入策略

- 轮询(按照消息尽量保证每一个分区的负载)策略，消息会均地分布到每个partition

  写入消息地时候，key为null的时候，默认使用的是轮询策略

- 随机策略（不使用）

- 按key写入策略，key.hash()%

- 自定义分区策略

> 乱序问题
>
> - 在Kafka中生产者是有写入策略，如果topic有多个分区，就会将数据分分散在不同的partition中存储
> - 当partition数量大于1的时候，数据（消息）会打散分布在不同的partion中
> - 如果只有一个分区，消息是有序的

#### 消费组Consumer Group Rebaleance机制

- 再均衡：在某些情况下，消费者组中的消费的分区会产生变化，会导致消费者分配不均匀（例如：有两个消费者消费3个，因为某个partition崩溃了，还有一个消费者当前没有分区要削峰），当Kafka Consumer Group就会启动rebalance机制，重新平衡这个 Consumer Group 内的消费者消费的分区分配。

- 触发时机

  - 消费者数量发生变化
    -  某个消费者crash
    - 新增消费者

  - topic的数量发生变化

    - 某个topic被删除

  - partition的数量发生变化

    -  删除partition

    -  新增partition

- 不良影响

  - 发生rebalance，所有的consumer将不再工作，共同来参与再均衡，直到每个消费者都已经被成功分配所需要消费的分区为止（rebalance结束）

#### 消费者的分配策略

分区分配策略：保障每个消费者尽量能够均衡地消费分区数据据，不能出现某个消费者分区的数量特别多，某个消费者消费的分区特别少

- Rangele分配策略（范围分配策略）
  - n：分区的数量/消费者数量
  - m：分区的数量%消费者数量
- 前m个消费者消费n+1个分区
- 剩余的消费者消费nge
- RoundRobin分配策略（轮询分配策略）
- 消费者挨个分配消费的分区
- Striky粘性分配策略
  - 在没有发生rebalance跟轮询分配策略是一致的
  - 发生了rebalance，轮询分配测率。重新走一遍轮询分配的过程。而粘性会保证跟上一次的尽量一致，只是将新的需要分配的分区，均匀的分配到现有可用的消费者中即可
  - 减少上下文的切换

### 副本的ACK机制

producer是不断地往Kafka中写入数据，写入数据会有一个返回结果，表示是否写入成功。这里对应有一个ACKs的配置。

- acks = 0：生产者只管写入、不管是否写入成功，可能会数据丢失。性能是最好的
- acks=1：生产者会等到leader分区写入成功后，返回成功，接着发送吓一跳
- acks=-1/all：确保消息写入到leader分区、还确保消息写入到对应副本都成功后，接着发送下一条性能是最差的

根据业务情况来选择ack机制，是要求性能最高，一部分数据丢失影响不大，可以选择0/1。如果要求数据一定不能丢失，就得配置为-1/all。

分区中是有leader和follower的概念，为了确保消费者消费的数据是一致的，只能从分区leafer去读写消息follower做的事情就是同步数据，Backup

### 高级API（High-Level API）、低级API（Low-level API）

- 高级APi就是直接让Kafka帮助管理、处理分配、数据
  - offset存储在ZK中
  - 由Kafka的rebalance来控制消费者分配的分区
  - 开发起来比较简单，无需开发者关注底层细节
  - 无法做到细粒度的控制
- 低级API：由编写的程序来控制逻辑
  - 自己来管理offset，可以将offset存储在ZK、MySQL、Redis、HBase、Flink的转台存储
  - 指定消费者拉取某个分区的数据
  - 可以做到细粒度的控制
  - 原有的Kafka的策略会失效，需要我们自己来实现消费机制

## Kafka原理

### leader和follower

- Kafka中的leader和follower 是相对分区有意义，不是相对broker
- Kafka在创建topic的时候，会尽量分配分区的leader在不同的broker中，其实就是负载均衡
- leader职责：读写数据
- follower职责：同步数据、参与选举（leader crash 之后，会选举一个follower重新称为分区的leader）
- 注意和ZooKeeper区分
- ZK的leader负责读、写、follower可以读取
  - ZK的leader负责读、写、followe可以读取
  - Kafka的leader负责读写、follower不能读取数据（确保每个消费者消费的数据是一致的），Kafka一个topic有多个分区leader，一样可以实现数据操作的负载均衡

###  AR\ISR\OR

- AR表示一个topic下的所有副本
- ISR：In Sync Replicas，正在同步的副本（可以理解为当前几个follower是存活的）
- OSR：Out of Sync Replicas，不在同步的副本
- AR = ISR+OSR

### leader选举

Controller：controller是Kafka集群的老大，是针对Broker的一个角色

- controller是高可用的，是通过ZK来进行选举

Leader：是针对partition的一个角色

- leader是通过ISR来进行快速选举

如果Kafka是基于ZK来进行选举，ZK的压力可能会比较大。例如：某个节点崩溃，这个节点上不仅仅有一个leader，是由不少的leader需要选举。通过ISR快速进行选举

leader的负载均衡

- 如果某个broker crash之后，就可能回导致partition的leader分布不均匀，就是一个broker上存在一个topic下不通partition的leader
- 通过以下指令，可以将leader分配到优先的leader对用的broker，确保leader是均匀分配的

```
bin/kafka-leader-election.sh --bootstrap-server node1:9092 --topic test --partition=2 --election-type preferred
```

### Kafka读写流程

写流程

- 通过ZooKeeper找partition对应的leader，leader是负责读写
  - producer开始写入数据
  - ISR里面的follower开始同步数据，并返回给leader ACK
  - 返回给producer ACK

- 读流程
  - 通过Zookeeper找partition对应的leader，leader是负责读的
  - 通过Zookeeper找到消费者对应的offset
  - 然后开始从offset往后顺序拉取数据
  - 提交offset（自动提交-每隔多少秒提交一次offset、手动提交--放入到事务中提交）

### Kafka的物理存储

- Kafka的数据组织结构
  - topic
  - partition
  - segment
    - .log数据文件
    - .index（稀疏索引）
    - .timeindex（根据时间做的索引）
- 深入了解数据的流程
  - 消费者的offset是一个针对partition全局offset
  - 可以根据这个offset找到segment段
  - 接着需要将全局的offset转换成segment的局部offset
  - 根据局部的offset，就可以从（.index稀疏索引）找到对应的数据位置
  - 开始顺序读取

### 消息传递语义性

Flink里面又对应的每种不同机制的保证，提供Exactly-once保障（二阶段事务提交方式）

- At-most once：最多一次（只管把数据消费到，不管有没有成功，可能有数据丢失）
- At-least once：最少一次（有可能会出现重复消费）
- Exactly once：仅又一次（事务性的保障，保证消息有且呗处理一次）

### Kafka消息不丢失

- broker消息不丢失：又副本relicas的存在，会不断的从leader中同步副本，所以，一个broker crash，不会导致数据丢失，除非只有一个副本。

- 生产者消息不丢失：ACK机制（配置成ALL/-1）、配置成0，或者1可能存在丢失

- 消费者消费不丢失，重点控制offset

  At-least once：一种数据可能会重复消费

  Exactly once：仅被一次消费

### 数据积压

- 数据挤压是指消费者因为一些外部IO、一些比较耗时的操作（Full GC —— Stop the world），就会造成消息在partition中一直存在得不到消费，就会产生数据积压
- 在企业中，我们要有监控系统，如果出现这种情况，需要尽快处理。虽然后续的Spark Streaming/Flink可以实现背压机制，但是数据库积累太多一定对实时系统的实时性是有影响的

## Kafka Eagle 安装

