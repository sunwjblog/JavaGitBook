# Kafka入门

### 什么事Kafka？

kayak的两个特性：

* 用于两个关系紧密的系统或应用之间的实时流管道传输
* 用于处理实时流数据

Kafak可以说事，面向数据流的生产、转换、存储、消费为整体的处理平台。

### Kafka基本概念

* 物理概念：物理层面的隔离，如数据库、服务器设备等
* 逻辑概念：代码/策略逻辑层面的概念。
* Producer：消费和数据的生产者，向Kafka的一个topic发布消息的进程/代码/服务
* Consumer：消息和数据的消费者，订阅数据（Topic）并且处理其发布的消息的进程/代码/服务
* Consumer Group：逻辑概念，对于同一个topic，会广播给不同的group，一个group中，只有一个consumer可以消费该消息。
* Broker：物理概念，Kafka集群中的每个Kafka节点
* Topic：逻辑概念，Kafka消息的类别，对数据进行区分、隔离
* Partition：物理概念，Kafka下数据存储的基本单元。一个Topic数据，会被分散存储到多个Partition，**每一个Partition事有序的**。
  * 每一个Topic被切分为多个Partitions
  * 消费者数目少于或等于Partition的数目
  * Broker Group中的每一个Broker保存Topic的一个或多个Partitions
  * Consumer Group中的仅有一个Consumer读取Topic的一个或多个Partitions，并且是唯一的Consumer

* Replication：同一个Partion可能会有多个Replica，多个Replica之间数据是一样的
  * 当集群中有Broker挂掉的情况，系统可以主动地使Replicas提供服务
  * 系统默认设置每一个Topic的repliaction系数为1，可以在创建Topic时单独设置
  * 特点
    * Replication的基本单位是Topic的Partition
    * 所有的读和写都从Leader进，Followers只是作为备份
    * Follower必须能够及时复制Leader的数据
    * 增加容错性与可扩展性
* Replication Leader：一个Partition的多个Replica上，需要一个Leader负责该Partition上与Produce和Consumer交互
* ReplicaMananger：负责管理当前broker所有分区和副本的信息，处理KafkaController发起的一些请求，副本状态的切换、添加/读取消息等。

### Kafka基本结构

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/Kafka基本结构.png)

### Kafka特点

* #### 分布式

  * 多分区
  * 多副本
  * 多订阅者
  * 基于ZooKeeper调度

* 高性能

  * 高吞吐量 每秒几十万
  * 低延迟
  * 高并发
  * 时间复杂度为O(1)

* 持久性与扩展性

  * 数据可持久性
  * 容错性
  * 支持在线水平扩展
  * 消息自动平衡

### Kafka应用场景

* 消息队列
* 行为跟踪
* 元信息监控
* 日志收集
* 流处理
* 事件源
* 持久性日志
