# Kafka

[toc]

## 1.基础概念

![在这里插入图片描述](https://blog-1257815336.cos.ap-nanjing.myqcloud.com/typora/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODg4NTY3,size_16,color_FFFFFF,t_70.png)

![在这里插入图片描述](https://blog-1257815336.cos.ap-nanjing.myqcloud.com/typora/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODg4NTY3,size_16,color_FFFFFF,t_70-20210602141410927.png)

![在这里插入图片描述](https://blog-1257815336.cos.ap-nanjing.myqcloud.com/typora/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODg4NTY3,size_16,color_FFFFFF,t_70-20210602141424057.png)

**Kafka**

Kafka 是一个分布式的消费引擎系统，主要功能是提供一套完备的消息发布与订阅解决方案

**Topic**

在 kafka 中，发布订阅的对象是`主题`，主题的划分主要依据业务的定义

**Producer**

在 kafka 当中，向主题发布消息的客户端应用称为 `生产者`，生产者通常持续不断地向一个或者多个主题发送消息

**Consumer**

而订阅这些主题消息的客户端应用程序就被称为消费者，消费者也能够同时订阅多个主题的消息。

**Broker**

Kafka 的服务端是由被称为 Broker 的服务进程构成，一个 Kafka 集群由多个 Broker组成，Broker 负责接收和处理客户端发送过来的请求，以及对其进行持久化。

由于 Kafka 是分布式的消息引擎，Broker 自然也就支持分散运行在不同的机器上。如果集群中某一个 Broker 挂了，其他 Broker 依然可以提供服务

**Partition**

Kafka的 Partition 的概念就类似于 Es 中的 shard 概念，也是将某个主题下的消息划分成多个分区，`每个分区都是一组有序的消息日志`，生产者生产的每条消息只会发送到一个分区中。

**副本机制**

Kafka会对每个分片的数据进行拷贝，这些拷贝出来的数据就叫做副本。

Kafka中定义了副本的两种类型：Leader 和 Follower 

Leader 负责对外提供服务，而 Follower 仅负责跟随 Leader 进行数据同步。

工作机制上：生产者总是向 Leader 写入消息，消费者总是从 Leader 读取消息；而追随者永远指向 Leader 发起同步请求，进行数据同步。

**持久化消息**

Kafka 使用消息日志来保存消息，一个日志就是磁盘上的一个可追加写文件。

由于是追加写入，顺序 IO 因此Kafka 的吞吐性能较好。

同时为了避免消息日志的大小不断增加，日志文件又被分成日志段，当写满一个日志段后，会将老日志段封存，等到合适的时机再删除。



**消费者组**

点对点模型(Peer to Peer，P2P) 和发布订阅模型。这里面的点对点指的是同一条消息只能被下游的一个消费者消费，其他消费者则不能染 指。在Kafka中实现这种P2P模型的方法就是引入了消费者组(Consumer Group)。所谓的消费者组，指的 是多个消费者实例共同`组成一个组来消费一个主题`。`这个主题中的每个分区都只会被组内的一个消费者实例 消费，其他消费者实例不能消费它。为什么要引入消费者组呢?主要是为了提升消费者端的吞吐量`。

> 消费者组里面的所有消费者实例不仅“瓜分”订阅主题的数据，而且更酷的是它们还能彼此协助。假设组内 某个实例挂掉了，Kafka能够自动检测到，然后把这个Failed实例之前负责的分区转移给其他活着的消费 者。这个过程就是Kafka中大名鼎鼎的“重平衡”(Rebalance)。
>
> 每个消费者在消费消息的过程中必然需要有个字段记录它当前消费到了分区的哪个位置上，这个字段就是消 费者位移(Consumer Offset)。注意，这和上面所说的位移完全不是一个概念。上面的“位移”表征的是 分区内的消息位置，它是不变的，即一旦消息被成功写入到一个分区上，它的位移值就是固定的了。而消费 者位移则不同，它可能是随时变化的，毕竟它是消费者消费进度的指示器嘛。另外每个消费者有着自己的消 费者位移，因此一定要区分这两类位移的区别。我个人把消息在分区中的位移称为分区位移，而把消费者端的位移称为消费者位移。



## 2.Kafka 多分片多副本机制的好处

1.Kafka 通过给定特定 Topic 指定多个 partition,而各个 Partition 可以分布在不同的 Broker上，并发吞吐更高

2.Partition 可以指定对应的 Replica 数，极大的提高了消息存储的安全性能，提高了容灾能力。

## 3.Kafka 如何保证消息的消费顺序

首先结合业务场景来说，客户中心目前对于消息并不要求顺序消费。

如果需要要求顺序消费的话，我们一般有两种做法：

1.首先 kafka 的消息都是存储在各个 topic 下的 多个partition 内的，且在单个partition内的消息顺序是可以保证有序的。如果需要保证某个主题的消息有序，可以将该主题的partition 设置为 1，这样就可以保证 topic 下的 消息最终都落到一个 Partition 下，且由于 partition 内的消息都是采用尾加法写入，partition 内的消息都是有序的。

2.也是比较推荐的做法，kafka 在发送消息的时候可以指定 topic\partition\key\value 4 个参数，所以生产者在发送消息的时候可以指定 partition，且让同一个 Key的消息都发送到同一个 partition。

## 4.Kafka 如何保证消息不丢失

### 生产者

生产者调用 send 方法发送消息后，可能因为网络问题并没有发送过去。

所以调用send 方法成功并不能保证消息一定发送成功了，为了保证消息发送的结果，我们可以调用带回调函数的 send 方法，根据回调，一旦出现消息提交失败的情况，就可以有针对性地进行处理。

同时记得设置 Producer 的 retires 重试次数，一般设置为 3-5，以及重试间隔。对于网络波动来说，可能时间一长，重试次数就用完了。

### 消费者

消费者在消费 partition 上的消息时，都会记录当前所在 partition 的消费进度，也就是偏移量 Offset ，偏移量 offet 标识当前消费者消费到的 Partition 所在的位置，同时消费者也是通过 Offset 来保证分区内的顺序性。

当消费者拉取到分区的某个消息后，消费者会自动提交 Offset。但也同时带来了个问题，由于客户中心消费的时候是开启多线程去消费，可能存在某个线程运行失败了，它负责的消息没有被成功处理，但位移已经被更新了。因此这条消息对于`consumer`而言实际上是丢失了。这里的关键就在自动提交`offset`，如何真正地确认消息是否真的被消费，再进行更新`offset`。

所以消息者可以关闭自动提交，通过业务逻辑判断当前消息确实消费成功后再去进行 offset的 commit



但关闭 offset 的自动提交也会带来另一个问题，即 同一个消息多次消费。



### 服务端丢失消息

服务端消息的丢失，可能存在的情况就是 leader partition 所在的Broker 突然挂了，且 follower partition 中的数据还没完全同步，就会造成消息丢失

这种可以通过调整 Broker 的参数进行包装

可以设置`acks = all`。`acks`是`Producer`的一个参数，代表“已提交”消息的定义。如果设置成`all`，则表明所有`Broker`都要接收到消息，该消息才算是“已提交”。

设置`min.insync.replicas > 1`。`Broker`端参数，控制消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。

设置`replication.factor >= 3`。这也是`Broker`端的参数。保存多份消息冗余，确保`replication.factor > min.insync.replicas`。如果两者相等，那么只要有一个副本离线，整个分区就无法正常工作了。推荐设置成`replication.factor = min.insync.replicas + 1`

### 实践配置

最后分享下`kafka`无消息丢失配置：

1. `producer`端使用`producer.send(msg, callback)`带有回调的`send`方法。
2. 设置`acks = all`。`acks`是`Producer`的一个参数，代表“已提交”消息的定义。如果设置成`all`，则表明所有`Broker`都要接收到消息，该消息才算是“已提交”。
3. 设置`retries`为一个较大的值。同样是`Producer`的参数。当出现网络抖动时，消息发送可能会失败，此时配置了`retries`的`Producer`能够自动重试发送消息，尽量避免消息丢失。
4. 设置`replication.factor >= 3`。这也是`Broker`端的参数。保存多份消息冗余，不多解释了。
5. 设置`min.insync.replicas > 1`。`Broker`端参数，控制消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在生产环境中不要使用默认值 1。确保`replication.factor > min.insync.replicas`。如果两者相等，那么只要有一个副本离线，整个分区就无法正常工作了。推荐设置成`replication.factor = min.insync.replicas + 1`。
6. 确保消息消费完成再提交。`Consumer`端有个参数`enable.auto.commit`，最好设置成`false`，并自己来处理`offset`的提交更新。



## 5.Kafka 如何保证不重复消费

导致 kafka 重复消费的原因

1.已经消费了数据，在提交 offset 之前 消费者挂掉了，rebalance 后新的消费者接替着从原来的 Offset去消费。

2.已经消费了数据，在提交 offset 之前,partition 因为处理时间太长超过了 session timeout 导致断开连接，rebalance 后新的消费者接替着从原来的 Offset去消费。

解决方法：

1.如果是需要入库的数据，可以现根据主键查询下，如果有了就不做入库操作。同时数据库也要做唯一索引约束

2.如果是需要 update 的数据，看业务上能否支持重复更新

3.如果业务上也要保证唯一性，那可以让上游生产者在发送消息的时候加个全局唯一的 id，然后每次消费后就存放在 redis 里，下次在遇到的时候可以先查，如果重复就不处理跳过。



## 6.Kafka 出现消息堆积、消费慢

Kafka 出现堆积的问题,首先查看 kafka 消息数量的监控，看是不是因为上游做了某些操作，比如业务问题导致用户在某个环节执行不下去了，大量流失。或者是公司做活动导致流失人数较之前上升。

第二，思考最近几次上线有没有修改消费者相关的代码。

第三，上线看消费日志，发现在堆积的消费中，有大量的重复消息。

同时发现在线程处理 kafka 消息的时候大量报出 commitFailed 错误，错误信息中提到 rebalanced，通过网上查阅发现可能是因为 消费者这边还在处理分区的消息的时候，分区却被 broker 回收了，因为 kafka 认为当前消费者已经死亡了。

出现的问题的原因是因为每次我们调用 poll 方法时，每次都拉取一批数据，等这批数据处理完成后才继续下一次轮训。

而我们这边设置的是手动提交，等待所有消息都提交完成处理后才会手动提交。

因此两次poll 之前拉取的时间超过了 max.poll.interval.ms 这个参数，默认是 300s。

如果客户端处理消息花费的时间超过了这个时间，broker 就会把消费者客户端移除掉，此时 Offset还没提交到 broker，分区被 rebalanced，导致下一次消费者又会从已经消费的地方继续消费，就出现了消费重复问题。

同时由于处理流程和最大拉取间隔的不匹配，下一个消费者也会出现超时，又 rebalanced，所以才会出现消费堆积情况。



**解决方法**

1. 使用Kafka时，消费者每次poll的数据业务处理时间不能超过kafka的max.poll.interval.ms，可以考虑调大超时时间或者调小每次poll的数据量。
   增加max.poll.interval.ms处理时长(默认间隔300s)

2. 可以考虑增强消费者的消费能力，使用线程池消费或者将消费者中耗时业务改成异步，并保证对消息是幂等处理

https://blog.csdn.net/qq_16681169/article/details/101081656





## 6.KafKa 技术选型

### 零拷贝技术

当Kafka客户端从服务器读取数据时，如果不使用零拷贝技术，那么大致需要经历这样的一个过程：

1.操作系统将数据从磁盘上读入到内核空间的读缓冲区中。

2.应用程序（也就是Kafka）从内核空间的读缓冲区将数据拷贝到用户空间的缓冲区中。

3.应用程序将数据从用户空间的缓冲区再写回到内核空间的socket缓冲区中。

4.操作系统将socket缓冲区中的数据拷贝到NIC缓冲区中，然后通过网络发送给客户端。

![img](https://blog-1257815336.cos.ap-nanjing.myqcloud.com/typora/format,png.png)



https://blog.csdn.net/zl1zl2zl3/article/details/107963699#:~:text=Kafka%E4%B8%BA%E4%BB%80%E4%B9%88%E9%80%9F%E5%BA%A6%E9%82%A3%E4%B9%88%E5%BF%AB,%E6%B5%B7%E9%87%8F%E6%95%B0%E6%8D%AE%E5%9C%BA%E6%99%AF%E5%B9%BF%E6%B3%9B%E5%BA%94%E7%94%A8%E3%80%82
