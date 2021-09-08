**RocketMQ 采用文件系统的方式来存储消息，消息的主要存储文件包括 CommitLog 文件、ConsumeQueue 文件、IndexFile 文件。**

-   **CommitLog** 是消息存储的物理文件，所有消息主题的消息都存储在 CommitLog 文件中，每个 Broker 上的 CommitLog 被当前机器上的所有 ConsumeQueue 共享。CommitLog 中的文件默认大小为 1G，可以动态配置；当一个文件写满以后，会生成一个新的 CommitLog 文件。所有的 Topic 数据是顺序写入在 CommitLog 文件中的。
-   **ConsumeQueue** 是消息消费的逻辑队列，消息达到 CommitLog 文件后将被异步转发到消息消费队列，供消息消费者消费，这里面包含 MessageQueue 在 CommitLog 中的物理位置偏移量 Offset，消息实体内容的大小和 Message Tag 的 hash 值。每个文件默认大小约为 600W 个字节，如果文件满了后会也会生成一个新的文件。
-   **IndexFile** 是消息索引文件，Index 索引文件提供了对 CommitLog 进行数据检索，提供了一种通过 key 或者时间区间来查找 CommitLog 中的消息的方法。在物理存储中，文件名是以创建的时间戳命名，固定的单个 IndexFile 大小大概为 400M，一个 IndexFile 可以保存 2000W 个索引。

## **消息存储的整体结构**

  

![](https://pic1.zhimg.com/80/v2-f0f5462d41799a90a02d84936b1c8b0c_720w.jpg)

  

RocketMQ 的消息存储采用的是混合型的存储结构，也就是 Broker 单个实例下的所有队列共用一个日志数据文件 CommitLog。这个是和 Kafka 又一个不同之处。为什么不采用 Kafka 的设计，针对不同的 Partition 存储一个独立的物理文件呢？这是因为在 Kafka 的设计中，一旦 Kafka 中 Topic 的 Partition 数量过多，队列文件会过多，那么会给磁盘的 IO 读写造成比较大的压力，也就造成了性能瓶颈。所以 RocketMQ 进行了优化，消息主题统一存储在 CommitLog 中。当然它也有它的优缺点。

-   优点在于：由于消息主题都是通过 CommitLog 来进行读写，ConsumerQueue 中只存储很少的数据，所以队列更加轻量化。对于磁盘的访问是串行化从而避免了磁盘的竞争。
-   缺点在于：消息写入磁盘虽然是基于顺序写，但是读的过程确实是随机的。读取一条消息会先读取 ConsumeQueue，再读 CommitLog，会降低消息读的效率。

## **消息发送到消息接收的整体流程**

1、Producer 将消息发送到 Broker 后，Broker 会采用同步或者异步的方式把消息写入到 CommitLog。RocketMQ 所有的消息都会存放在 CommitLog 中，为了保证消息存储不发生混乱，对 CommitLog 写之前会加锁，同时也可以使得消息能够被顺序写入到 CommitLog，只要消息被持久化到磁盘文件 CommitLog，那么就可以保证 Producer 发送的消息不会丢失。

  

![](https://pic1.zhimg.com/80/v2-223b8eca06cc5fd9ad28a855d1910b08_720w.jpg)

  

2、CommitLog 持久化后，会把里面的消息 Dispatch 到对应的 Consume Queue 上，Consume Queue 相当于 Kafka 中的 Partition，是一个逻辑队列，存储了这个 Queue 在 CommitLog 中的起始 Offset，log 大小和 MessageTag 的 hashCode。

  

![](https://pic3.zhimg.com/80/v2-a077178e56a8c2bfb7e7e6f2b14c91a6_720w.jpg)

  

3、当消费者进行消息消费时，会先读取 ConsumerQueue，逻辑消费队列 ConsumeQueue 保存了指定 Topic 下的队列消息在 CommitLog 中的起始物理偏移量 Offset，消息大小、和消息 Tag 的 HashCode 值。

  

![](https://pic4.zhimg.com/80/v2-1f4bd9d5f2388c5e779a82bf7d45cd63_720w.jpg)

  

4、直接从 ConsumerQueue 中读取消息是没有数据的，真正的消息主体在 CommitLog 中，所以还需要从 CommitLog 中读取消息。

  

![](https://pic4.zhimg.com/80/v2-e88e30839fd2cb1f28a49af5a0fae053_720w.jpg)

  

**消息刷盘**  

  

![](https://pic3.zhimg.com/80/v2-4797f54a5c52c71db542d4920ecda7d2_720w.jpg)

  

(1) 同步刷盘：如上图所示，只有在消息真正持久化至磁盘后RocketMQ的Broker端才会真正返回给Producer端一个成功的ACK响应。同步刷盘对MQ消息可靠性来说是一种不错的保障，但是性能上会有较大影响，一般适用于金融业务应用该模式较多。

(2) 异步刷盘：能够充分利用OS的PageCache的优势，只要消息写入PageCache即可将成功的ACK返回给Producer端。消息刷盘采用后台异步线程提交的方式进行，降低了读写延迟，提高了MQ的性能和吞吐量。