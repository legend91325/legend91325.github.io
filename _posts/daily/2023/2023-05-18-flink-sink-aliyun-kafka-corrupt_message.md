## 背景：

这段实时任务上阿里云 EMR过程中，使用到了阿里云的kafka 托管服务，记录下使用阿里云kafka托管服务中踩过的坑以及选择kafka类型的时候需要注意的点，供大家借鉴

## Flink sink Kafka 异常排错

### 异常
一个比较简单的fliink 写kafka的任务，当下游使用阿里云kafka的时候，会报如下异常：

```java
Caused by: org.apache.kafka.common.KafkaException: Could not add partitions to transaction due to errors: {xxxxxxxxx-8=CORRUPT_MESSAGE}  
at org.apache.kafka.clients.producer.internals.TransactionManager$AddPartitionsToTxnHandler.handleResponse(TransactionManager.java:1235)  
at org.apache.kafka.clients.producer.internals.TransactionManager$TxnRequestHandler.onComplete(TransactionManager.java:1074)  
at org.apache.kafka.clients.ClientResponse.onComplete(ClientResponse.java:109)  
at org.apache.kafka.clients.NetworkClient.completeResponses(NetworkClient.java:569)  
at org.apache.kafka.clients.NetworkClient.poll(NetworkClient.java:561)  
at org.apache.kafka.clients.producer.internals.Sender.maybeSendAndPollTransactionalRequest(Sender.java:425)  
at org.apache.kafka.clients.producer.internals.Sender.runOnce(Sender.java:311)  
at org.apache.kafka.clients.producer.internals.Sender.run(Sender.java:244)  
at java.lang.Thread.run(Thread.java:750)
```

### 排查定位

因为整个使用过程中，flink环境也进行了迁移适配，所以影响因素比较多，一时不好从哪里下手，所以找了套自建的kafka集群，大版本尽量保持一致，用新flink环境进行了实验，发现可以正确写入，**无异常报错**，这样基本可以确定是阿里云Kafka 版本本身特殊特性导致的，这个时候定位问题就比较容易了。

> 这里教大家一个经验，如果使用云产品，出现一些异常，第一时间先不用去搜索，而是去看官方的云产品文档，一般会把自家的产品常见的问题都会列出来。这样是最近直接的解决方案。

回到这个问题，我们看下阿里云官方给出的场景问题列表以及解决方案：
[# 使用消息队列Kafka版时客户端的报错及解决方案](https://help.aliyun.com/document_detail/124136.htm?spm=a2c4g.171530.0.0.3f9277c5M0BHas#concept-124136-zh)

发现了和我们异常匹配的错误：

>
**报错信息：**
>
`CORRUPT_MESSAGE`
>
**报错原因：**
>
>-   如果是云存储引擎：客户端版本大于等于3.0时，自动开启幂等功能， 但云存储不支持幂等功能
  >  
>-   如果是Local存储引擎：发送compact消息， 但未传递key值。
>
**解决方案**：
>
>-   如果是云储存引擎：设置`enable.idempotence=false`。
  >  
>-   如果是Local存储引擎：消息添加key值。


### 问题迷途

报错信息完全符合，但是我们的客户端版本是2.4.1 ，服务器端版本 2.6.2，同时我们用的时**阿里云Kafka标准版**，通过控制平台查看，确实topic 是**云存储引擎**  ，所以第一时间去确认我们kafka客户端是否有配置```enable.idempotence=false```

查看flink 任务中，kafka客户端配置日志：
```


```java
2023-05-18 11:22:15.110 [ForkJoinPool.commonPool-worker-8] INFO org.apache.kafka.clients.producer.ProducerConfig - ProducerConfig values:

    acks = 1

    batch.size = 16384

    bootstrap.servers = [xxxxxxxx:9092, xxxxxxxx:9092, xxxxxxxx:9092]

    buffer.memory = 33554432

    client.dns.lookup = default

    client.id =

    compression.type = none

    connections.max.idle.ms = 540000

    delivery.timeout.ms = 120000

    enable.idempotence = false

    interceptor.classes = []

    key.serializer = class org.apache.kafka.common.serialization.ByteArraySerializer

    linger.ms = 0

    max.block.ms = 60000

    max.in.flight.requests.per.connection = 5

    max.request.size = 1048576

    metadata.max.age.ms = 300000

    partitioner.class = class org.apache.kafka.clients.producer.internals.DefaultPartitioner

    receive.buffer.bytes = 32768

    reconnect.backoff.max.ms = 1000

    reconnect.backoff.ms = 50

    request.timeout.ms = 30000

    retries = 2147483647

    retry.backoff.ms = 100

    transaction.timeout.ms = 60000

    transactional.id = Source: KafkaSource -> TableFilter -> TableMap -> Sink: KafkaSink-2919cb34cac4f6c182f15e882575d8d2-13

    value.serializer = class org.apache.kafka.common.serialization.ByteArraySerializer
    省略 .....

```

发现我们的客户端版本配置中，已经设置了```enable.idempotence=false```，难道这个报错是其他问题引起的吗？ 

这个时候我们剥茧抽丝，先看看kafka客户端 ```enable.idempotence``` 具体含义，官方2.6.0版本给出的解释：
> [enable.idempotence](https://kafka.apache.org/26/documentation.html#enable.idempotence)

>When set to 'true', the producer will ensure that exactly one copy of each message is written in the stream. If 'false', producer retries due to broker failures, etc., may write duplicates of the retried message in the stream. Note that enabling idempotence requires `max.in.flight.requests.per.connection` to be less than or equal to 5, `retries` to be greater than 0 and `acks` must be 'all'. If these values are not explicitly set by the user, suitable values will be chosen. If incompatible values are set, a `ConfigException` will be thrown.

个人理解就是幂等性，就是需要生产者需要精准一致的写入记录，同时ack 需要设置为 all 。

>这个时候再回过来看我们的flink任务， 我们flink任务设置了**精准一致**语义，flinkProducerKafka 是开启事务的，所以我理解我们的flink是开启了事务，会不会和这个有关系呢？
>

### 最终定位

带着这个疑问，我又重新排查了flink 日志，发现了这句信息：

> [ForkJoinPool.commonPool-worker-8] INFO org.apache.kafka.clients.producer.KafkaProducer - [Producer clientId=producer-Source: KafkaSource -> TableFilter -> TableMap -> Sink: KafkaSink-2919cb34cac4f6c182f15e882575d8d2-12, transactionalId=Source: KafkaSource -> TableFilter -> TableMap -> Sink: KafkaSink-2919cb34cac4f6c182f15e882575d8d2-12] Overriding the default acks to all since idempotence is enabled.

我们看到 **Overriding the default acks to all since idempotence is enabled.**  这句话的意思是idempotence 是enable了！！！，但是我们明明设置了false啊， 下面就是深扒了源码了：


```java
// org.apache.kafka.clients.producer.KafkaProducer#configureAcks
private static short configureAcks(ProducerConfig config, boolean idempotenceEnabled, Logger log) {
        boolean userConfiguredAcks = false;
        short acks = (short) parseAcks(config.getString(ProducerConfig.ACKS_CONFIG));
        if (config.originals().containsKey(ProducerConfig.ACKS_CONFIG)) {
            userConfiguredAcks = true;
        }

        if (idempotenceEnabled && !userConfiguredAcks) {
            log.info("Overriding the default {} to all since idempotence is enabled.", ProducerConfig.ACKS_CONFIG);
            return -1;
        }

        if (idempotenceEnabled && acks != -1) {
            throw new ConfigException("Must set " + ProducerConfig.ACKS_CONFIG + " to all in order to use the idempotent " +
                    "producer. Otherwise we cannot guarantee idempotence.");
        }
        return acks;
    }
```


```java
//org.apache.kafka.clients.producer.KafkaProducer#newSender
// 471 line
short acks = configureAcks(producerConfig, transactionManager != null, log);
```

```java
//org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer#beginTransaction
    @Override
    protected FlinkKafkaProducer.KafkaTransactionState beginTransaction()
            throws FlinkKafkaException {
        switch (semantic) {
            case EXACTLY_ONCE:
                FlinkKafkaInternalProducer<byte[], byte[]> producer = createTransactionalProducer();
                producer.beginTransaction();
                return new FlinkKafkaProducer.KafkaTransactionState(
                        producer.getTransactionalId(), producer);
            case AT_LEAST_ONCE:
            case NONE:
                // Do not create new producer on each beginTransaction() if it is not necessary
                final FlinkKafkaProducer.KafkaTransactionState currentTransaction =
                        currentTransaction();
                if (currentTransaction != null && currentTransaction.producer != null) {
                    return new FlinkKafkaProducer.KafkaTransactionState(
                            currentTransaction.producer);
                }
                return new FlinkKafkaProducer.KafkaTransactionState(
                        initNonTransactionalProducer(true));
            default:
                throw new UnsupportedOperationException("Not implemented semantic");
        }
    }

```

**根据源码，可以看出如果是开启了事务，就是算开启了幂等性了，ack 也是被设置为了all了**

### 解决方案

问题终于定位到了，因为一般我们使用flink 任务在要求数据准确的场景下都要开启 **精准一次**语义，这个时候kafkaProdcer 是开启事务的，但是因为阿里云Kafka 标准版托管服务，竟然不支持**幂等性和事务性**，导致会报 **CORRUPT_MESSAGE** 错误，知道了具体问题原因就好解决了。

根据阿里云官方[存储引擎对比](https://help.aliyun.com/document_detail/123277.html?spm=a2c4g.120676.0.i4)
![aws emr](/assets/blog/2023/flink_kafka_corrupt_message/aliyun_kafka_diff.png)

我们可以看出来，推荐的云存储引擎，竟然**不支持幂等性和事务性**，同时更严重的事，因为内部是用的阿里云的自己**云盘算法**，在集群重启或者宕机时，会导致**极少数乱序**的可能性，真是让我大为惊奇，这还是推荐的选配方案啊，真是大坑的存在。
正常使用托管服务，如果不好好做调研的话很容易掉进这个坑的，各位使用的时候一定要结合自己使用场景，慎重选择，别因为云服务本身特性，导致自己数据错乱，得不偿失。

好了，知道具体原因了，那么解决方案就是铲掉现有标准版kafka集群，重建专业版kafka集群（价格上更高了，同时分区数也是1:3消耗量），手动创建topic 设置为local存储（自动声明的还是云存储，有点不能理解，后续多了很多手工的工作量）。


## 总结

个人也是用了很多云产品组件了，这种普遍使用的开源组件（大数据基本Kafka组件是必备的），做到了云服务产品之后，推荐的选配型号竟然和开源版差异这么大，也是很迷啊。建议在阿里云官方在创建Kafka集群页面上，至少要有充足的提示，让购买者能够充分权衡利弊。 

再多说一句，就好比今年阿里云的香港机房事件，如果默认oss的选配是推荐多可用区部署，而不是但可用区的话，是不是就不会有这么严重的事件了呢？ 当然我也就是个普通的云产品使用者，这个牢骚也就是算从使用者方面来感叹的。从我个人的理解，大众的云服务产品的教育普及还在初级教育阶段，很多使用者是没有关注细节的，拿来就用的比比皆是，所以云厂商不单要是从价格上竞争，还是要多考虑容灾情况，提前帮助像我一样的小白们，多多考虑。

好了 今天就告一段落，我们下期再见~~~


### 参考资料
[# Kafka 事务性之幂等性实现](https://matt33.com/2018/10/24/kafka-idempotent/#Client-%E5%B9%82%E7%AD%89%E6%80%A7%E6%97%B6%E5%8F%91%E9%80%81%E6%B5%81%E7%A8%8B)
[# 存储引擎对比](https://help.aliyun.com/document_detail/123277.html?spm=a2c4g.120676.0.i4)
[# enable.idempotence](https://kafka.apache.org/26/documentation.html#enable.idempotence)
[# 使用消息队列Kafka版时客户端的报错及解决方案](https://help.aliyun.com/document_detail/124136.htm?spm=a2c4g.171530.0.0.3f9277c5M0BHas#concept-124136-zh)