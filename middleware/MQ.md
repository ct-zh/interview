# Message Queue
> mq用的比较多的是 rabbitMQ、rocketMQ和kafka
## 消息队列问题
### 是否用过消息中间件?你们的用途是什么?为什么这样选型?你知道消费者组的概念吗?
consumer group是kafka提供的可扩展且具有容错性的消费者机制。组内有多个消费者或消费者实例(consumer instance)，它们共享一个group ID。组内的所有消费者协调在一起来消费订阅主题(subscribed topics)的所有分区(partition)。每个分区只能由同一个消费组内的一个consumer来消费。
- consumer group下可以有一个或多个consumer instance，consumer instance可以是一个进程，也可以是一个线程;
- group.id是一个字符串，唯一标识一个consumer group
- consumer group下订阅的topic下的每个分区只能分配给某个group下的一个consumer(当然该分区还可以被分配给其他group)

## 哪些业务场景使用了消息队列？为什么不能使用线程池开启多线程，可同样达到异步的效果？
列举几个常见场景:
- 秒杀系统: 大量秒杀请求进来, 先代码内部预扣库存,再将数据写到消息队列,让消费者慢慢去拉数据创建订单;
- 推送系统: 触发给用户推送消息的逻辑, 数据写到消息队列里面, 消费者慢慢发消息;

消息队列与协程/多线程的区别:
- 应用系统解耦
- 流量削峰

缺点:
- 消息丢失问题;
- 消息重复问题;
- 消息顺序问题;
- 消息丢失导致的一致性问题;

## 消息重复问题、消息丢失问题以及消息顺序怎么解决?
- 消息重复: 消费者做幂等设计;

    首先我们应该从上游就做消息去重处理，但是我们不能100%相信上游系统一定可靠.比如下单发红包的功能, 可以根据用户订单号或者流水号之类的做强幂等，每成功操作一次就记录下来，即使消息重复了，我只要判断同一个订单号已经发过红包了，后续我们就不会再做任何操作了。

- 消息丢失:sender设计,提供两个api:`SendMsg(msg bytes[])`(发送消息),`SendCallback()`;receiver设计,提供两个api:`RecvCallback(msg bytes[])`与`SendAck()`;流程如下:
    - sender使用`SendMsg`发送消息给MQ;
    - MQ消息落地,调用`SendCallback`将应答消息发给sender;(保证MQ必定收到消息)
    - SendMsg在未收到Callback时timer会重发消息;一般采用指数退避的策略，先隔x秒重发，2x秒重发，4x秒重发，以此类推
    - receiver使用`RecvCallback`从MQ收到消息;
    - receiver使用`SendAck`向MQ申明已经收到消息了; MQ删除对应数据;MQ在未收到ack时会不停重发; (确保消息必达)
    - *sender会重发很多次消息, receiver会收到很多重复消息, 因此接口的幂等一定要做好*

- 消费顺序问题: 

    想要实现消息有序就要牺牲性能/可靠性。
    - Producer：让生产端同步发送消息，消息1确定发送成功后再发送消息2，不能异步，保证消息顺序入队。
    - 服务端：Producer -> MQ Server -> Consumer 一对一关系，一对一服务，这肯定能保证消息是按照顺序消费的，那么问题来了：Producer -> MQ Server -> Consumer任意一个环节出现问题，那肯定整个链路都阻塞了.单通道模型性能成为瓶颈。
    - topic不分区：意思就是让同一个topic主题都入一个队列，在分布式环境下如果同一个topic进入多个分区，那多个分区之间肯定无法保证消息顺序了。
    - Consumer：保证消费端是串行消费，禁止使用多线程。
    但是这些方法都会牺牲掉系统的性能和稳定性，顺序性问题非要使用MQ来做，那也没有太好的办法了。

- 如何做到消息不分区:
    - RocketMQ：RocketMQ提供了MessageQueueSelector队列选择机制，我们可以把 Topic 用Hash取模法，相同Topic的Hash值肯定是一样的，让同一个Topic 同一个队列中，再使用同步发送，这样就能保证消息在一个分区有序了。
    - Kafka： Kafka可以把 max.in.flight.requests.per.connection 参数设置成1，这样就可以保证同一个topic在同一个分区内了。

## kafka
### 对Kafka的了解, 为什么选择用Kakfa
生产者，消费者，消费者组，主题，发布订阅，消息确认等。吞吐量高, Kafka与CDH结合好.Kafka除了可以解耦架构，还可以用来做异步存储，化解瞬间高流量。

### Kafka消息的事务性怎么实现的
Kafka Producer：有三种ack层级，分别是不确认，分区主节点确认，分区所有节点确认
kafka Consumer: 开发者保证ack，参考[JStorm](https://github.com/dooonabe/no-class-is-an-island/blob/master/article/Stream/JStorm.md)的Acker机制

### 假设事务提交的消息没有发到broker上面,会怎么处理
Kafka生产者开启最高等级的ack开关

### Kafka消息如何顺序
Kafka主题每个分区内是有序的，但是分区间并不保证有顺。开发者可以将需要保证顺序的数据发送到同一分区中。例如：将同一用户的数据发送到同一个分区。

## 为什么选择Kafka呢？ 可以简述下Kafka架构中比较重要的关键字吗？比如 Partition，Broker，你都是怎么理解的？
- Kafka 支持批量拉取消息;
- 支持多种发送场景:发送并忘记 同步发送  异步发送 + 回调函数;
- 分布式可高可扩展;
- 吞吐量比较大

为什么吞吐量比较大?
- 利用了磁盘连续读写性能远远高于随机读写的特点，内部采用消息的批量处理，zero-copy 机制，数据的存储和获取是本地磁盘顺序批量操作，具有 O (1) 的复杂度，消息处理的效率很高。
- 并发，将一个 topic 拆分多个 partition， kafka 读写的单位是 partition，因此，将一个 topic 拆分为多个 partition 可以提高吞吐量。但是，这里有个前提，就是不同 partition 需要位于不同的磁盘（可以在同一个机器）。如果多个 partition 位于同一个磁盘，那么意味着有多个进程同时对一个磁盘的多个文件进行读写，使得操作系统会对磁盘读写进行频繁调度，也就是破坏了磁盘读写的连续性。

关键字:
- Producer: 生产者;
- Consumer: 消费者;
- Topic: 每条发布到 MQ 集群的消息都有一个类别，这个类别被称为 topic，可以理解成一类消息的名字。所有的消息都以 topic 作为单位进行归类。
- Partition: Kafka 物理上分区的概念，每个 Topic 会分散在一个或多个 Partition。一个 Topic 的数据太大了，就分成小片，Kafka 为分区引入多副本模型，副本之间采用 “一个 leader 多 follower” 的设计，通过多副本实现故障自动转移，保证可用性。
- Broker: 可以理解成一个服务器的节点，集群包含一个或多个服务器，这种服务器被称为 broker。对应用来说，生产者把消费发出去了，就不管了。消费者慢条斯理地按照自己的速率来消费。这段时间可能有大量消息产生，消费者压力还是在一定范围内。做生产者和消费者之间解耦的就是一个缓存服务 broker。

ps. 常用 Kafka 开源管理工具：
- [Kafka Manager](https://github.com/yahoo/kafka-manager)
- [Kafka Lens](https://github.com/kafka-lens/kafka-lens)
- [Kafka Monitor](https://github.com/linkedin/kafka-monitor)

《深入理解 Kafka：核心设计与实践原理》