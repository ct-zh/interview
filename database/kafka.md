1. 对Kafka了解吗? Kafka消息的事务性怎么实现的
    Kafka Producer：有三种ack层级，分别是不确认，分区主节点确认，分区所有节点确认
    kafka Consumer: 开发者保证ack，参考[JStorm](https://github.com/dooonabe/no-class-is-an-island/blob/master/article/Stream/JStorm.md)的Acker机制

2. 假设事务提交的消息没有发到broker上面,会怎么处理
    Kafka生产者开启最高等级的ack开关

3. 对Kafka的了解, 为什么选择用Kakfa
    生产者，消费者，消费者组，主题，发布订阅，消息确认等。吞吐量高, Kafka与CDH结合好.Kafka除了可以解耦架构，还可以用来做异步存储，化解瞬间高流量。

4. Kafka消息如何顺序
    Kafka主题每个分区内是有序的，但是分区间并不保证有顺。开发者可以将需要保证顺序的数据发送到同一分区中。例如：将同一用户的数据发送到同一个分区。

