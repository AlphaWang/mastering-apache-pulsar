# 第 4 章：Pulsar 原理

So far, we’ve discussed the motivation for using a system like Apache Pulsar, the historical context in which Pulsar was created, and some companies that use Pulsar to power their systems. Now we have sufficient context to pull the covers from Pulsar and explore the components and, more important, why they work together. We’ll start by looking at each of Pulsar’s components (see [Figure 4-1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pulsarapostrophes_components_include_no)): namely Pulsar brokers, Apache BookKeeper, and Apache ZooKeeper. Then we’ll take a look at a standard technology used across all three of these projects: the Java programming language and the Java virtual machine.

到目前为止，我们讨论了使用 Apache Pulsar 这类系统的动机、构建 Pulsar 的历史背景，以及一些使用 Pulsar 为驱动其系统的公司。 现在我们有了足够的上下文，可以揭开 Pulsar 的面纱，探索其中各个组件，更重要的是，为什么这些组件能协同工作。 我们将从查看 Pulsar 的每个组件开始（见图 4-1]）：即 Pulsar Broker、Apache BookKeeper 和 Apache ZooKeeper。 然后我们会看一下在这三个项目中使用的标准技术：Java 编程语言和 Java 虚拟机。



![Pulsar’s components include nodes, Apache BookKeeper, and Apache ZooKeeper.](../img/mapu_0401.png)

*图 4-1. Pulsar 组件包括 Broker节点、Apache BookKeeper 和 Apache ZooKeeper。*

# Broker

As noted earlier, Pulsar’s modularity allows the system to separate its responsibilities and select the best technology to handle each one. One of Pulsar’s responsibilities is to provide an interface so that publishers and subscribers can connect to it.

Pulsar brokers handle this as well as the following tasks (see [Figure 4-2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pulsar_nodes_have_an_underlying_impleme)):

- Temporary management of topic data storage
- Communication with Apache BookKeeper and ZooKeeper
- Schema validation
- Inter-broker communication
- Runtime environments for Pulsar Functions and Pulsar IO

如前所述，Pulsar 的模块化允许系统分离其职责，并选择最佳技术来处理每个职责。 Pulsar 的职责之一是提供一个接口，以便发布者和订阅者可以连接到它。

这个职责是由 Pulsar Broker 实现的，此外它还处理以下任务（参见 图 4-2）：

- 主题数据存储的临时管理
- 与 Apache BookKeeper 和 ZooKeeper 的通信
- Schema 验证
- Broker 之间的通信
- Pulsar Function 和 Pulsar IO 的运行时环境

![Pulsar nodes have an underlying implementation in Java on the Java Virtual Machine. Pulsar Functions and Pulsar IO are also implemented in Java. Pulsar supports several HTTP and TCP points for communication within the cluster.](../img/mapu_0402.png)

*图 4-2. Pulsar Broker 节点的底层实现基于 Java 虚拟机上的 Java 语言。 Pulsar Function 和 Pulsar IO 也是基于 Java 实现。 Pulsar 支持多个 HTTP 和 TCP 端点用于在集群内进行通信。*



Let’s take a closer look at Pulsar brokers.

下面让我们来仔细看看 Pulsar Broker。



## 消息缓存

Pulsar brokers are stateless, in that they do not store any data on the Pulsar broker disks that are used in the message lifecycle. Pulsar is unique among message brokers in this approach, as most similar systems couple the storage and retrieval of messages in some way. Being stateless has advantages as well as disadvantages. The disadvantages are that another system is required to take on state management and some abstractions are required to translate from Pulsar’s storage needs to the storage system. The advantages are that storage requirements are separate from compute requirements and that a more fault-tolerant storage layer results.

Pulsar Broker 是无状态的，Pulsar Broker 磁盘中不会存储消息生命周期中使用的任何数据。这相对于其他消息 Broker 来说是独一无二的，因为大多数类似的系统都以某种方式将消息的存储和检索耦合在一起。无状态既有优点也有缺点。缺点是需要另一个系统来进行状态管理，并且需要一些抽象来将 Pulsar 的存储需求转移到存储系统。优点则是存储需求与计算需求是分开的，并且会产生更容错的存储层。



If Pulsar brokers were responsible for storing the state of topics on the broker, a number of questions would arise regarding how to store data on the brokers and how to handle failure scenarios.

如果 Pulsar Broker 也负责存储主题状态，则会出现许多问题，例如如何在代理上存储数据，以及如何处理故障场景。



Since we’re just beginning our journey with Pulsar, let’s keep it simple and explore just the following three considerations:

- Storing data
- Adding new nodes to the cluster
- Removing nodes from the cluster

由于我们刚刚开始使用 Pulsar，让我们保持简单，只探讨以下三个注意事项：

- 存储数据
- 向集群添加新节点
- 从集群中删除节点



In [Chapter 3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#pulsar), we discussed what storing data in a distributed system looks like in terms of storage and retrieval for a low-volume system. A high-volume system has even more things to consider when it comes to how data is distributed across nodes and how events like losing a node impact the entire system. Instead of taking on the complexity of understanding the storage problem, Pulsar chose to rely on Apache BookKeeper for storage and to use the brokers as stateless orchestrators of the storage.

在[第 3 章](./ch03-pulsar.md)中，我们讨论了分布式系统中的数据存储，在低容量系统中是如何存储和检索的。而大容量系统则需要考虑更多因素，这会涉及到数据如何跨节点分布，以及发送节点丢失这样事件后会如何影响整个系统。 Pulsar 没有承担理解存储问题的复杂性，而是选择依赖 Apache BookKeeper 进行存储，并将 Broker 用作无状态的存储协调器。



Pulsar uses an abstraction on top of BookKeeper, called a managed ledger. The managed ledger works as a bridge between the messages that Pulsar brokers need to store and the ledgers in BookKeeper (covered later in this chapter). You can think of ledgers as the highest storage abstraction in BookKeeper. The managed ledger is an API that keeps track of the ledger sizes and states and when it’s time to start a new ledger.

Pulsar 在 BookKeeper 之上使用了一个抽象，称为 Managed Ledger。Managed Ledger 在 Pulsar Broker 与 BookKeeper 直接充当桥梁作用，连接 Pulsar Broker需要存储的消息和 BookKeeper 中的 Ledger（本章稍后介绍）。你可以将 Ledger 看做是 BookKeeper 中的最高层存储抽象。Managed Ledger 是一个 API，用于跟踪 Ledger 大小和状态，以及决定何时启动新 Ledger。



[Figure 4-3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#in_this_apache_pulsar_clustercomma_book) shows a typical topology of a Pulsar topic. Broker 1 is responsible for topic reads and writes. For reads（原文有误）, it writes to all the BookKeeper instances (main servers, or bookies) that are part of the ensemble for the topic; for reads, it requests data from the leader for that ledger. The managed ledger manages that interface. Does this mean that for every write Pulsar broker has to retrieve data from the bookies? Not exactly. Pulsar brokers have a managed ledger cache that allows some messages to be cached on the broker for a consumer.

图 4-3 展示了 Pulsar 主题的一个典型拓扑。 Broker 1 负责主题的读写。对于写请求，它写入作为主题 ensemble 的所有 BookKeeper 实例（称为主服务器或 bookie）；对于读请求，它向该 Ledger 的领导者请求数据。Manged Ledger 管理该接口。这是否意味着每次读取 Pulsar Broker 都必须从 Bookie 检索数据？不完全是。 Pulsar Broker 有一个 Managed Ledger 缓存，允许在 Broker 上为消费者缓存一些消息。

![In this Apache Pulsar cluster, bookies store data from the topic.](../img/mapu_0403.png)

*图 4-3. 在 Apache Pulsar 集群中，bookies 存储来自主题的数据。*



In a streaming context, each message needs to be written to BookKeeper. Instead of writing to BookKeeper and reading from it for an active consumer, Pulsar brokers can simply tail the latest events directly to an active consumer. This avoids making round trips to BookKeeper, as depicted in [Figure 4-4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_pulsar_broker_can_tail_the_latest_eve).

在流式上下文中，每条消息都需要写入 BookKeeper。 对于最新写入的事件，Pulsar Broker 无需为活跃消费者写入 BookKeeper 再从中读取出来，而是可以简单地将最新事件直接返回给活跃消费者。 这避免了读写 BookKeeper 的往返时间，如图 4-4 所示。



![A Pulsar broker can tail the latest events directly to an active consumer.](../img/mapu_0404.png)

*图 4-4. Pulsar Broker 可以将最新事件直接返回给活跃消费者。*

It’s important to remember that even though the managed ledger can cache values for consumers that are subscribed to the topic, the cache is only a cache (see [Figure 4-5](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#the_managed_ledger_cache_is_a_configura)). Caches are ephemeral and are created and destroyed easily. They are not supposed to be permanent data stores, as data that is stored in a cache is a convenience but also a potential headache. Fortunately, Pulsar brokers have a limited scope in which they cache data. In Chapters [5](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch05.html#consumers) and [6](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch06.html#producers) you’ll learn more about Pulsar’s messaging lifecycle.

重要的是要记住，即使 Managed Ledger 可以为订阅主题的消费者缓存消息，但是缓存终究是缓存（参见 图 4-5)。缓存是临时的，很容易被创建和销毁。它们不应该是永久的数据存储，因为存储在缓存中的数据虽然很遍历，但也可能令人头疼。幸运的是，Pulsar Broker 缓存数据的范围是有限的。在第 [5](./ch05-consumers.md) 章和第 [6](./ch06-producers.md) 章中，你将了解更多有关 Pulsar 消息生命周期信息。



![The managed ledger cache is a configurable cache kept by the Pulsar broker. It stores a ledger of data stored in BookKeeper and keeps an interface to write to BookKeeper.](../img/mapu_0405.png)

*图 4-5. Managed Ledger 缓存一个可配置缓存，由 Pulsar Broker 保存。用于缓存存储在 BookKeeper 中的 Ledger 数据，并保留一个向 BookKeeper 写入数据的接口。*



## BookKeeper 与 ZooKeeper 之间的通信

As discussed in this chapter’s introduction, Pulsar nodes work in conjunction with BookKeeper and ZooKeeper as the messaging platform’s backbone. Not surprisingly, Pulsar brokers need to communicate with ZooKeeper and BookKeeper for topic management and other configuration values. How and when this communication takes place is fully managed by the Pulsar brokers. It’s worth taking some time to better understand when brokers communicate with BookKeeper and ZooKeeper.

正如本章介绍中所述，Pulsar Broker 节点与 BookKeeper 和 ZooKeeper 一起工作，成为消息平台的骨干。毫不奇怪，Pulsar Broker 需要与 ZooKeeper 和 BookKeeper 进行通信，来进行主题管理以及读写配置值。这种通信的具体发生方式以及发生时间完全由 Pulsar Broker 管理。值得花一些时间来好好了解 Broker 何时与 BookKeeper 和 ZooKeeper 进行通信。



ZooKeeper stores all metadata related to the Pulsar cluster. This includes metadata about which broker is the leader for a topic, configuration values for service discovery, and other administrative data. Much of the data stored in ZooKeeper is cached on the Pulsar nodes, and there is a configuration-driven lifecycle about when to pull new data from ZooKeeper. Communicating with ZooKeeper is a constant part of Pulsar’s lifecycle.

ZooKeeper 存储着与 Pulsar 集群相关的所有元数据。包括哪个 Broker 是主题的 Leader、服务发现的配置值、以及其他管理数据。 ZooKeeper 中存储的大部分数据都缓存在 Pulsar Broker 节点上，另外有配置驱动的生命周期决定何时从 ZooKeeper 拉取新数据。与 ZooKeeper 进行通信是 Pulsar 生命周期中的一个固定部分。



As discussed in previous sections, BookKeeper is the storage engine in Pulsar. All message data is stored in Pulsar. Every message stored and retrieved from Pulsar requires communication with BookKeeper. BookKeeper’s communication interfaces are covered in more detail in [Chapter 12](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch12.html#operating_pulsar).

如前几节所述，BookKeeper 是 Pulsar 的存储引擎。所有消息数据都存储在 Pulsar 中。从 Pulsar 存储和检索的每条消息都需要与 BookKeeper 通信。BookKeeper 的通信接口在 [第 12 章](./ch12-operating_pulsar.md)中有更详细的介绍。



## Schema 验证

Schema validation is the process of ensuring that new messages published to a Pulsar topic adhere to a predefined shape. To ensure that a message adheres to a schema, Pulsar brokers work with the Pulsar schema registry to perform that validation. The lifecycle of schema validation is covered in [Chapter 6](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch06.html#producers); however, the responsibility of ensuring schema is significant and falls squarely on the brokers, so we’ll discuss it briefly here.

Schema 验证是确保发布到 Pulsar 主题的新消息符合预定义格式的过程。为了确保消息符合 Schema 要求，Pulsar Broker 与 Pulsar Schema 注册表一起协同执行验证。Schema 验证的生命周期在 [第 6 章](./ch06-producers.md) 中会有介绍；然而确保 Schema 要求的责任重大，且完全由 Broker 承担，所以我们将在这里简要讨论一下。



Brokers handle schema validation in two key ways. First, they are the point of ownership for schemas as they relate to topics. Brokers answer the following questions:

- Does this topic have a schema associated with it?
- What is the schema associated with the topic?
- Does this schema require that new messages adhere to the schema?

Broker 以两种关键方式处理 Schema 验证。首先，Schema 是与主题相关的，因此 Broker 拥有对 Schema 的所有权。Broker 将回答以下问题：

- 该主题是否有与之关联的 Schema？
- 与该主题相关的 Schema 是什么？
- 该 Schema 是否要求新消息遵循该 Schema？



Also, brokers can ensure validation of in-flight messages. Schema validation is an important part of end-to-end messaging systems, and Pulsar brokers serve this purpose, among others.

此外，Broker 可以确保对正在发送的消息进行验证。Schema 验证是端到端消息系统的重要组成部分，Pulsar Broker 的一项职责就是进行 Schema 验证。



## Broker 之间的通信

As mentioned previously, brokers are responsible for the reads and writes of specific topics. It is possible for a client to request data from a broker that is not responsible for that topic. What happens in this case? [Figure 4-6](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#broker_one_is_not_the_leader_for_topic) depicts this. Each broker uses metadata stored in ZooKeeper to determine whether it is the leader (the one responsible for the topic) and, if it is not the leader, who the leader is. The broker may route the client to the correct broker to start publishing (or retrieving) messages.

如前文所述，每个 Broker 负责特定主题的读取和写入。 客户端有可能向不负责该主题的 Broker 请求数据。 这种情况下会发生什么？ 图 4-6 描述了这一点。 每个 Broker 使用存储在 ZooKeeper 中的元数据来确定它是否是 Leader（负责该主题的 Broker），如果它不是 Leader，那么 Leader 是谁。 Broker 可以将客户端路由到正确的Broker 以开始发布（或检索）消息。



![Broker 1 is not the leader for Topic A, and therefore redirects the producers to the correct topic.](../img/mapu_0406.png)

*图 4-6. Broker 1不是主题 A 的 Leader，因此将生产者重定向到正确的 Broker。*



## Pulsar Function 与 Pulsar IO

At the beginning of this section, I stressed the importance of modularity in Pulsar’s design. In the sections that followed, you learned how much responsibility falls on the brokers. It may have occurred to you that perhaps Pulsar’s design could be more modular. It’s important to remember two things when considering modularity. First, does it make sense to remove the responsibilities from the brokers and put them elsewhere? And second, would moving those responsibilities elsewhere necessarily improve Pulsar as far as reliability and scalability are concerned? As a general rule, the answer to both questions is no. The exceptions to this rule are Pulsar IO and Pulsar Functions.

在本节的开头，我强调了模块化在 Pulsar 设计中的重要性。在接下来的章节中，你了解了 Broker 承担哪些责任。你可能已经想到，也许 Pulsar 的设计可以更加模块化。在考虑模块化时，必须记住两件事。首先，将 Broker 的责任转移到其他地方是否有意义？其次，将这些职责转移到其他地方是否一定会改善 Pulsar 的可靠性和可扩展性？一般来说，这两个问题的答案都是否定的。这个规则的例外是 Pulsar IO 和 Pulsar Function。



Pulsar as a project provides some easy methods for getting started with the base Pulsar brokers as well as extensions such as Pulsar Functions and Pulsar IO. You can use Pulsar Functions or Pulsar IO for a new Pulsar user without additional overhead or difficulty. The limiting factor to this convenience is that brokers are the primary source for throughput in Pulsar. How many messages a cluster can ingest per second is highly influenced by a broker’s availability. If the broker is busy processing Pulsar Functions or Pulsar IO tasks, it will impact the entire system’s performance.

Pulsar 不仅提供了一些简单的方法来上手使用 Pulsar Broker 的核心功能，还提供了 Pulsar Function 和 Pulsar IO 等扩展功能。你可以为新 Pulsar 用户使用 Pulsar Function 或 Pulsar IO，而无需额外的开销或困难。这种便利性的限制因素源于 Broker 是 Pulsar 吞吐量的主要来源。集群每秒可以接收多少消息很大程度上受 Broker 的可用性影响。如果 Broker 忙于处理 Pulsar Function 或 Pulsar IO 任务，那么整个系统的性能都会收到影响。



In many cases this performance degradation won’t be problematic, but for sufficient scale, moving your Pulsar IO or Pulsar function to another cluster would be an improvement. Fortunately, Pulsar provides a mechanism for precisely this.

在多数情况下，这种性能下降不会有问题，但当集群规模足够大时，将 Pulsar IO 或 Pulsar Function 转移到另一个集群就是一个改进。幸运的是，Pulsar 正好提供了这种机制。



# Apache BookKeeper

[Apache BookKeeper](https://oreil.ly/dOm7v) is a general-purpose data storage system. BookKeeper, like Pulsar and ZooKeeper, was developed at Yahoo! in the 2010s to meet the following requirements:

- Write and read latencies of < 5 ms
- Durable, consistent, and fault-tolerant data storage
- Read data as it is written
- Provide an interface for real-time and long-term storage

[Apache BookKeeper](https://oreil.ly/dOm7v) 是一个通用的数据存储系统。 BookKeeper 与 Pulsar 和 ZooKeeper 一样，是由雅虎在 2010 年代开发的，用以满足以下要求：

- < 5 ms 的写入和读取延迟
- 持久、一致和容错的数据存储
- 对数据边写边读
- 提供实时和长期存储的接口



BookKeeper is an ambitious project, aimed at building primitives for storage that could work for a wide number of projects and long into the future. BookKeeper is written in Java and heavily utilizes Apache ZooKeeper (which we’ll cover later in this chapter). [Figure 4-7](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#bookies_are_servers_where_data_is_store) shows BookKeeper’s architecture. The main servers are called bookies, and they can be arranged as a cluster (ZooKeeper can be arranged in the same way). The bookies contain an underlying storage system called a ledger.

BookKeeper 是一个雄心勃勃的项目，旨在构建存储原语，以期可以适用于大量项目。 BookKeeper 是用 Java 编写的，并且大量使用了 Apache ZooKeeper（我们将在本章后面介绍）。 图 4-7 展示了 BookKeeper 的架构。主要的服务器称为 Bookie，它们可以组合成一个集群（ZooKeeper 也可以按照相同的方式组合）。Bookie 包含一个称为 Ledger 的底层存储系统。

![Bookies are servers where data is stored (on a ledger). Apache ZooKeeper (ZK) manages service discovery and coordination among the bookies.](../img/mapu_0407.png)

*图 4-7。 Bookie 是（在 Ledger 上）存储数据的服务器。 Apache ZooKeeper (ZK) 管理 Bookie 之间的服务发现和协调。*



How do you go about building a system with the performance requirements and durability promised by Apache BookKeeper? The breakdown of requirements from a high level is as follows:

- A simple semantic for storing data
- A fault-tolerant way to distribute the storage of data across nodes
- An easy way to recover from any individual node failure

要构建一个像 Apache BookKeeper 那样高性能及高持久性的系统，应该如何着手呢？ 高层次的需求可以细分如下：

- 一种用于存储数据的简单语义
- 一种高度容错的跨节点分布数据存储的方式
- 一种从任何单个节点故障中恢复的简单方法



Starting with the first requirement, Apache BookKeeper implements an append-only log called a *ledger.* A ledger consists of arbitrary data called *entries.* A sequence of ledgers is called a *stream* (see [Figure 4-8](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_high-level_view_of_bookkeeper_storage))*.*

从第一个要求开始，Apache BookKeeper 实现了一个称为 *Ledger* 的仅附加日志。Ledger 由称为 *Entry* 的任意数据组成。Ledger 的序列称为 *stream*（见图 4-8)。



![A high-level view of BookKeeper storage. The stream is a collection of ledgers, and ledgers are composed of smaller entries.](../img/mapu_0408.png)

*图 4-8。 BookKeeper 存储的高层视图。 Ledger 的集合成为 Stream，Ledger 则由较小的 Entry 组成。*



Creating entries and ledgers with the Apache BookKeeper Java client is simple as well. BookKeeper has two Java APIs: the BookKeeper Ledger API and the Advanced Ledger API. The BookKeeper Ledger API is lower level, focused on allowing the user to interact directly with ledgers. The Advanced Ledger API provides some additional features that give the user more fine-grained control around quorum configuration (covered shortly) and other aspects of transaction safety for BookKeeper. For our purposes, we’ll do a few things with the Ledger API to illustrate what it might look like to interact with BookKeeper directly.

使用 Apache BookKeeper Java 客户端创建 Entry 和 Ledger 也很简单。 BookKeeper 有两个 Java API：BookKeeper Ledger API 和 Advanced Ledger API。 BookKeeper Ledger API 是较低级别的 API，专注于让用户直接与 Ledger 交互。 Advanced Ledger API 提供了一些额外的功能，让用户可以更细粒度地控制仲裁配置（稍后将介绍），以及配置 BookKeeper 交易安全等其他方面。 出于我们的目的，我们将使用 Ledger API 来说明如何直接与 BookKeeper 交互。



We’ll perform these operations:

- Create a new BookKeeper client
- Create a ledger
- Write entries to the ledger
- Close the ledger
- Reopen the ledger
- Read all entries

我们将执行这些操作：

- 创建一个新的 BookKeeper 客户端
- 创建一个 Ledger
- 将 Entry 写入 Ledger
- 关闭 Ledger 
- 重新打开 Ledger
- 读取所有 Entry



To begin, take a look at the following code:

首先，看一下下面的代码：

```java
// Create a client object for the local ensemble.
BookKeeper bkc = new BookKeeper("localhost:2181");

// A password for the new ledger
byte[] ledgerPassword = /* a sequence of bytes */;

// Create a new ledger and get identifier
LedgerHandle lh = bkc.createLedger(BookKeeper.DigestType.MAC, ledgerPassword);
long ledgerId = lh.getId();

// Create a buffer for ten-byte entries
ByteBuffer entry = ByteBuffer.allocate(10);

int numberOfEntries = 100;

// Add entries to the ledger, then close the ledger
for (int i = 0; i < numberOfEntries; i++){
      entry.putInt(i);
      entry.position(0);
      lh.addEntry(entry.array());
}
lh.close();

// Reopen the ledger
lh = bkc.openLedger(ledgerId, BookKeeper.DigestType.MAC, ledgerPassword);

// Read all entries
Enumeration<LedgerEntry> entries = lh.readEntries(0, numberOfEntries - 1);

while(entries.hasMoreElements()) {
      ByteBuffer result = ByteBuffer.wrap(ls.nextElement().getEntry());
      Integer retrEntry = result.getInt();

    // Get all entries printed
    System.out.println(String.format("Result: %s", retrEntry));
}

// Close the ledger and stop the client
lh.close();
bkc.close();
```

The preceding code gives us significant context to move on to the second question we laid out, which is how do we store segments across multiple nodes?

上述代码为我们提供了重要的背景，使我们可以继续讨论我们提出的第二个问题，即我们如何跨多个节点存储数据分片？



Apache BookKeeper uses quorum-based replication to manage the problem of distributing data across nodes. The protocol has some complexities, but we can focus on the main aspect of it to better understand how it relates to Pulsar topics.

Apache BookKeeper 使用基于仲裁的复制协议来处理跨节点分布数据的问题。该协议有一些复杂性，但我们可以把重点放在其主要方面，以便更好地理解它与 Pulsar 主题的关系。



The BookKeeper protocol requires the following for every ledger:

- The ensemble size (`E`)

  Represents the number of bookies the ledger will be stored on.

- The quorum write size (`Q_w`)

  Represents the number of nodes each entry will be written to.

- The quorum acknowledgment (ack) size (`Q_a`)

  Represents the number of nodes an entry must be acknowledged by.

BookKeeper 协议对每个 Ledger 都有以下要求：

- Ensemble Size（`E`）

  表示将要存储 Ledger 的 Bookie 的数量。

- 仲裁写入大小 (`Q_w`)

  表示每个 Entry 将被写入的节点数。

- 仲裁确认 (ack) 大小 (`Q_a`)

  表示 Entry 必须被确认的节点数。
  
  

In general, the ensemble has to be greater than or equal to the quorum write size. This is a sensible requirement because you can’t have more bookies that accept a new ledger than there are in the entire cluster. Also, the quorum ack size must be less than or equal to the quorum write size. This also makes sense because, at a minimum, you want every write node to acknowledge the write of a new entry, but reducing the number of nodes required to acknowledge new entries might increase overall performance without having any impact on the redundancy or safety of the data.[^i] 

通常，Ensemble 必须大于或等于仲裁写入大小。这是一个合理的要求，因为接受新 Ledger 写入的 Bookie 数目不可能比整个集群的所有 Bookie 数量还多。此外，仲裁确认大小必须小于或等于仲裁写入大小。这也是有道理的，因为最差情况下你希望每个写入 Entry 的节点都确认，但减少确认新 Entry 所需的节点数量可以提高整体性能，而不会对数据冗余或数据安全性产生任何影响。[^i]

[^i]:  如果您的写入大小非常大，那么即使一个节点无法接收写入，数据也会被足够多的节点复制，以便在节点故障时进行恢复。



It may be helpful to walk through a few examples. [Figure 4-9](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_bookkeeper_quorum_has_four_ledgers) depicts a BookKeeper configuration in which the ensemble size is 3, the quorum write size is 3, and the quorum ack size is 3. Each new ledger is written to every bookie and every bookie must acknowledge their writing.

通过几个例子来讲解可能更有帮助。图 4-9 描述了一个 BookKeeper 配置，其中 Ensemble 大小为 3，Quorum Write 大小为 3，Quorum Ack 大小为 3。每个新的 Ledger 都会写入每个 Bookie，每个 Bookie 都必须确认他们的写入。

![This BookKeeper quorum has four ledgers (represented as squares) and an ensemble size of 3, a quorum write size of 3, and a quorum ack size of 3.](../img/mapu_0409.png)

*图 4-9。 该 BookKeeper Quorum 有四个正方形的 Ledger，Ensemble 大小为 3，Quorum Write 大小为 3，Quorum Ack 大小为 3*



[Figure 4-10](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_bookkeeper_example_has_an_ensemble) depicts an ensemble of 5, with a quorum write size of 3 and a quorum ack size of 3. In this scenario, each ledger is written to only three bookies.

图 4-10 展示了一个大小为 5 的 Ensemble，quorum write size 为 3，quorum ack size 为 3。在这种情况下，每个 Ledger 只写入三个 Bookie。

![This BookKeeper example has an ensemble of 5, with a quorum write size of 3 and a quorum ack size of 3.](../img/mapu_0410.png)

*图 4-10. 该 BookKeeper Quorum为 5，quorum write 大小为 3，quorum ack 大小为 3。*



Now that you know a bit about BookKeeper’s storage internals, let’s take a step back and examine some of the new terminology and ideas I just introduced.

现在你对 BookKeeper 的存储内部原理有了一些了解，让我们退一步来看看我刚刚介绍的一些新术语和想法。



Quorums are used a lot in organizational contexts, but in distributed system contexts, a quorum is simply a group of processes. For BookKeeper, quorums are used for ledger management, but they are also used as a mechanism for keeping track of which bookies are the leaders for a given segment. We won’t get into the topic of leader election here, but [Figure 4-11](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#leader_election_in_bookkeeperdot_bookie) provides a decent high-level overview of the concept.

Quorum 在组织环境中被大量使用，但在分布式系统环境中，Quorum 只是一组进程。对于 BookKeeper，Quorum 用于 Ledger 管理，也用于跟踪哪些 Bookie 是给定数据分片的领导者。这里我们不谈 Leader 选举的话题，但是图4-11 对这个概念给出了一个高层概述。



![Leader election in BookKeeper. Bookies are leaders for segments and can be removed or changed by a new leader election event.](../img/mapu_0411.png)

*图 4-11. BookKeeper 中的领导者选举。 Bookies 是数据分片的领导者，可以通过新的领导者选举删除或更改。*



Now that you understand the basics of storage, you’ll notice that BookKeeper provides the basic building blocks for storing data and keeping it safe. A single bookie may contain a fragment of a ledger, and that fragment is replicated across several bookies. [Figure 4-12](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#fragments_left_parenthesisparts_of_ledg) shows what this might look like for three ledgers across three bookies.

现在你了解了存储的基础知识，你将注意到 BookKeeper 提供了用于存储数据和保证数据安全的基本构件。 一个 Bookie 可能包含一个 Ledger 片段，并且该片段被复制到多个 Bookie 中。图 4-12 展示了跨三个 Bookie 的三个Ledger 的情况。

![Fragments (parts of ledgers) are stored across different bookies in a BookKeeper ensemble.](../img/mapu_0412.png)

*图 4-12。 片段是 Ledger 的一部分，存储在 BookKeeper Ensemble 中的不同 Bookie 节点中。*



The design and storage primitives in BookKeeper make it suitable for complex ledgers that can span an ever-increasing number of bookies, as depicted in [Figure 4-13](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_bookkeeper_ensemble_has_several_fr).

BookKeeper 中的设计和存储原语使其适用于复杂 Ledger，可以分布到越来越多 Bookie 上，如图 4-13 所示。

![This BookKeeper ensemble has several fragments spanning bookies.](../img/mapu_0413.png)

*图 4-13. 该 BookKeeper Ensemble 有几个跨 Bookie 的片段。*



You may be wondering how this system benefits Apache Pulsar. Topics (the fundamental message storage paradigm in Pulsar) are implemented on BookKeeper. In a system where every message is critical, BookKeeper makes it virtually impossible[^ii] to lose any messages. Additionally, BookKeeper ledgers are an append-only log. As such, it’s the perfect primitive for storing data for an event streaming system, as discussed in [Chapter 2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#event_streams_and_event_brokers).

你可能好奇这样的系统对 Apache Pulsar 有什么好处。主题是 Pulsar 中的基本消息存储范式主题，它是在 BookKeeper 上实现的。在每条消息都至关重要的系统中，BookKeeper 几乎不可能[^ii] 丢失任何消息。此外，BookKeeper Ledger 是仅附加的日志。因此，它完美适配存储事件流系统数据的场景，如 [第 2 章](./ch02-event_streams_and_event_brokers.md) 所述。

[^ii]: 配置不当的集群可能会丢失数据。



[There is a lot more to BookKeeper](https://oreil.ly/JgpTL), but hopefully this section provided some insight into its elegance. Storage of Pulsar messages is one use case for BookKeeper; let’s explore a few others to solidify our understanding.

[BookKeeper 还有很多其他功能](https://oreil.ly/JgpTL)，但希望本节能让大家对它的优雅设计有一些了解。 Pulsar 消息的存储是 BookKeeper 的一个使用场景；让我们探索其他一些使用场景来巩固我们的理解。



## 预写日志

A write-ahead log (WAL) is used to provide atomicity and durability in a database system. This book isn’t about databases, but the WAL is a critical concept to understand in order to grasp the value of BookKeeper. If you think about a database table, you can perform actions such as inserts, updates, selects, and deletes. When you perform an insert, update, or delete, the database writes your desire to perform that action to a log (see [Figure 4-14](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#in_this_write-ahead_log_implementationc)). The database can then check against the log to validate it performed the action intended by the user. WALs are not only useful for ensuring guarantees in databases; they are also used for change data capture (CDC). Pulsar IO utilizes WALs in databases to perform CDC (we’ll cover this in [Chapter 7](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch07.html#pulsar_io-id000027)).

预写日志 (WAL) 用于在数据库系统中提供原子性和持久性。本书不是关于数据库的，但为了理解 BookKeeper 的价值，WAL 是一个重要的概念。假设有一个数据库表，可以执行诸如插入、更新、查询和删除等操作。当执行插入、更新或删除操作时，数据库会将执行该操作的愿望写入日志（参见 图 4-14)。然后数据库可以对照日志验证是否执行了用户预期的操作。 WAL 不仅有助于确保数据库的保证，还用于变更数据捕获 (CDC)。 Pulsar IO 利用数据库中的 WAL 来执行 CDC（我们将在 [第 7 章](./ch07-puslar_io.md) 介绍这一点）。



![In this write-ahead log implementation, each new event is written to a log before it is executed on the underlying database storage engine.](../img/mapu_0414.png)

*图 4-14. 在预写日志实现中，每个新事件在底层数据库存储引擎上执行之前都会写入日志。*



BookKeeper’s durability properties, fault tolerance, and scalability make it the right choice for a WAL implementation. Additionally, BookKeeper can scale separately from the database if needed, providing modularity and loose coupling.

BookKeeper 的持久性、容错性和可扩展性使其成为 WAL 实现的正确选择。此外，如果需要，BookKeeper 可以与数据库分开扩展，提供模块化和松耦合。



## Message Storing

Every messaging system has some implementation for storing messages temporarily. A message broker’s value is in the reliable transport of messages, after all. How we implement that storage will have some downstream consequences on how the data can be used later. For systems like Pulsar, Kafka, and Pravega, message durability is paramount. BookKeeper’s model of ledger storage is the perfect abstraction for storing an event stream (see [Figure 4-15](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_topic_has_sequential_data_written)).

![This topic has sequential data written to BookKeeper ledgers.](../img/mapu_0415.png)

*Figure 4-15. This topic has sequential data written to BookKeeper ledgers.*



Following are some of the properties that make BookKeeper a good solution for event stream data:

- Append-only logging
- Highly durable
- Easily distributed

## Object/Blob Storage

Object stores allow the storage of arbitrarily large objects for future retrieval. Systems like Amazon S3, Google Cloud Storage, and Azure Blob Storage are popular because they offer a simple API to store and retrieve items from the cloud. In cloud systems, object storage is used for storing images, arbitrary files on behalf of users, and large data lakes. Implementing an object store requires elasticity, or the ability to add new nodes to the cluster without disrupting ongoing operations. It also requires fault tolerance; if nodes in the cluster fail, there should be a reliable backup in the cluster somewhere. In addition, it requires the ability to store and retrieve objects of all kinds. Apache BookKeeper can perform all these tasks and can perform them well. BlobIt is an object store built on top of BookKeeper. It allows for the storage of arbitrarily large objects, and all of the storage is managed with BookKeeper. A user can send a CSV file to BlobIt and the file will be stored on BookKeeper as depicted in [Figure 4-16](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#you_can_store_a_csv_file_on_bookkeeper).

![You can store a CSV file on BookKeeper for use as a general-purpose object store.](../img/mapu_0416.png)

*Figure 4-16. You can store a CSV file on BookKeeper for use as a general-purpose object store.*



While BookKeeper can store arbitrarily large data, the complexity in using it as an object store is in managing the movement of the data to and from bytes. BlobIt relies on the distributed and fault-tolerant nature of BookKeeper, and adds value by making an Amazon S3–compliant API.

## Pravega

Pravega is a distributed messaging system that has a lot of similarities to Pulsar. Developed at Dell, Pravega builds on the concept of a stream as the fundamental building block for storage. Pravega uses BookKeeper in a similar way to Pulsar (see [Figure 4-17](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pravega_architecturedot_similar_to_apac)): storing all topic and cursor data. Like Pulsar, BookKeeper enables Pravega to scale storage and message throughput independently, and it provides durability for and fault tolerance within a Pravega cluster.

![Pravega architecture. Similar to Apache Pulsar, Pravega uses BookKeeper for long-term storage, uses ZooKeeper for distributed coordination, and has some responsibilities that are owned by the Pravega servers.](../img/mapu_0417.png)

*Figure 4-17. Pravega architecture. Similar to Apache Pulsar, Pravega uses BookKeeper for long-term storage, uses ZooKeeper for distributed coordination, and has some responsibilities that are owned by the Pravega servers.*



An additional interesting tidbit about Pravega is that its use cases extend beyond just event streaming data (an area that Pulsar is focused on). Pravega is also suitable for streaming video data and large files. As mentioned previously*,* you can store any data on BookKeeper; the challenges lie in how that data is presented and how end users interact with it.

## Majordodo

Majordodo is a resource manager that handles the scheduling of bespoke workloads on ephemeral clusters. Majordodo tracks the resources used in a cluster, the available resources in a cluster, and other metadata about jobs running in a cluster (see [Figure 4-18](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_majordodo_clusterdot_bookkeeper_manag)). [Majordodo](https://oreil.ly/kh1T4) utilizes BookKeeper ledgers to track the starting, running, and completion of jobs on the cluster. Since BookKeeper provides low read and write latencies, scheduling workloads is a novel but worthy use. Majordodo is developed and maintained by Diennea, a technology company that helps build scalable digital brand solutions.

![A Majordodo cluster. BookKeeper manages data storage for each node so that any node in the cluster can pick up work or distribute existing work.](../img/mapu_0418.png)

*Figure 4-18. A Majordodo cluster. BookKeeper manages data storage for each node so that any node in the cluster can pick up work or distribute existing work.*



In the preceding sections, we spent a lot of time talking about the importance and use cases for BookKeeper. ZooKeeper works in conjunction with BookKeeper and plays a different but equally important role in Pulsar and in the wider software ecosystem. ZooKeeper is discussed in the next section.

# Apache ZooKeeper

Apache ZooKeeper was developed at Yahoo! in the late 2000s, and the Apache Software Foundation made it an open source platform in 2011. ZooKeeper implements the system described in the 2006 paper “The Chubby Lock Service for Loosely-Coupled Distributed Systems” by Mike Burrows,[^iii] a distinguished engineer at Google. The paper explains why Google needed Chubby to manage its sprawling internal systems and provides some high-level descriptions of its implementation.

[^iii]: Mike Burrows, “Chubby lock service for loosely-coupled distributed systems,” *OSDI ’06: Proceedings of the 7th Symposium on Operating Systems Design and Implementation* (November 2006): 24.



Chubby provides tools for distributed configuration management, service discovery, and a two-phase commit implementation. Chubby is a proprietary service used within Google, and the paper provides a peek at how Google handled a standard set of distributed system problems. With some light shed on how to approach these problems, Yahoo! implemented Apache ZooKeeper.

ZooKeeper provides an open source implementation suitable for coordinating distributed systems. ZooKeeper’s primary requirements are:

- Performance
- Fault tolerance
- Reliability

By meeting these requirements, ZooKeeper is suitable for implementing several distributed system algorithms, including Paxos and Raft. Another standard implementation on top of ZooKeeper is the two-phase commit protocol. The two-phase commit ensures atomicity, or that all nodes have a shared understanding of the system’s current state in a distributed system. The two-phase commit is illustrated in [Figure 4-19](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#in_this_two-phase_commitcomma_a_new_cha).

![In this two-phase commit, a new change is accepted by the leader and immediately sent to followers as part of a single transaction.](../img/mapu_0419.png)

*Figure 4-19. In this two-phase commit, a new change is accepted by the leader and immediately sent to followers as part of a single transaction.*



In Pulsar, ZooKeeper manages the BookKeeper configuration and distributed consensus, and stores metadata about Pulsar topics and configuration. It plays an integral role in all Pulsar operations, and replacing it would be difficult. It’s worth diving into a few examples of use cases for ZooKeeper to understand its importance to Pulsar.

## Naming Service

One common way systems integrate with Apache ZooKeeper is as a naming service. A naming service maps network resources to their respective addresses. In a system with many nodes, keeping track of their identity and their place in the network can be tricky. Apache Mesos uses ZooKeeper for this purpose (among others). In a Mesos cluster, ZooKeeper stores every node, their status, and a leader or follower. If nodes need to coordinate, ZooKeeper can be used as a lookup. ZooKeeper serves this purpose in Apache Pulsar as well, as depicted in [Figure 4-6](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#broker_one_is_not_the_leader_for_topic) earlier in this chapter.

## Configuration Management

Apache Pulsar has about 150 configuration values that are tunable by Pulsar operators. Each value changes the underlying behavior of Pulsar, ZooKeeper, or BookKeeper. Some of those configurations impact the publishing and consumption of messages in the Pulsar cluster. Pulsar brokers store their configuration in ZooKeeper because a reliable and highly available place to retrieve and store those configurations is paramount.

In some ways, ZooKeeper is a safe, distributed storage engine. As [Figure 4-20](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_zookeeper_cluster_can_store_multiple) shows, ZooKeeper can keep track of named keys and values.

![A ZooKeeper cluster can store multiple configuration values.](../img/mapu_0420.png)

*Figure 4-20. A ZooKeeper cluster can store multiple configuration values.*

## Leader Election

Leader election is the process of choosing a leader for a specific set of responsibilities in a distributed system. In Apache Pulsar, a broker is the leader of a topic (or one or more partitions in a partitioned topic). If that broker goes offline, a new broker is elected the leader of that same topic or partition(s). Building on both the naming service use case and the configuration use case, ZooKeeper can provide a reliable building block for implementing leader election. It keeps track of leaders, knows where they are in the cluster, and can be called on to implement new leaders in the future, as depicted in [Figure 4-21](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#in_this_leader-follower_modelcomma_the).

![In this leader-follower model, the values are tracked across ZooKeeper nodes.](../img/mapu_0421.png)

*Figure 4-21. In this leader-follower model, the values are tracked across ZooKeeper nodes.*

## Notification System

The final ZooKeeper use case that we’ll cover is that of a notification system. In [Chapter 1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch01.html#the_value_of_real-time_messaging), you learned how notifications can help patients in a hospital receive better care. The most important aspects of a notification system are the timely delivery of notifications and the guaranteed delivery of notifications. If you miss a notification about engagement on a tweet you sent late last night, that isn’t a world-stopping event. However, if you miss a notification to renew your driver’s license, you may be arrested the next time you are pulled over. We’ve discussed how ZooKeeper serves as a high-quality naming service. The same qualities that make ZooKeeper a good naming service make it an excellent notification system. Namely, we can ensure that the system state is shared by all parties, and that if a party doesn’t have that notification, we can quickly determine it using ZooKeeper. [Figure 4-22](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#the_commit_protocol_used_in_zookeeper_i) provides a high-level view of this concept.

![The commit protocol used in ZooKeeper is useful as a notification protocol.](../img/mapu_0422.png)

*Figure 4-22. The commit protocol used in ZooKeeper is useful as a notification protocol.*

## Apache Kafka

Apache Kafka is a distributed messaging system suitable for event streaming. It has broad adoption because of its thoughtful API design and scalability characteristics. Developed at LinkedIn and made freely available in 2014, Kafka provides the building blocks for event management and real-time systems at companies around the world. As of version 2.5, Kafka utilizes Apache ZooKeeper for configuration management and leader election. ZooKeeper plays a critical role in fault tolerance and message delivery in a Kafka cluster. [Figure 4-23](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_apache_kafka_cluster_has_three_bro) depicts a Kafka cluster with Apache ZooKeeper.

![This Apache Kafka cluster has three brokers, and ZooKeeper for coordination and leader election.](../img/mapu_0423.png)

*Figure 4-23. This Apache Kafka cluster has three brokers, as well as ZooKeeper for coordination and leader election.*

Interestingly, the Kafka project removed the requirement of ZooKeeper in a Kafka cluster as of version 2.8 and replaced the ZooKeeper responsibilities with a Raft consensus implementation within the cluster itself. [Figure 4-24](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_kafka_cluster_has_three_nodes_with) depicts this change.

![This Kafka cluster has three nodes with their own consensus algorithm, known as KRaft (Kafka Raft).](../img/mapu_0424.png)

*Figure 4-24. This Kafka cluster has three nodes with their own consensus algorithm, known as KRaft (Kafka Raft).*

## Apache Druid

Apache Druid is a real-time analytics database originally developed by Metamarkets in 2011 and made freely available through the Apache Software Foundation in 2015. Druid powers the analytics suite of companies like Alibaba, Airbnb, and [Booking.com](http://booking.com/). Unlike SQL-on-Anything engines[^iiii] such as Presto and Apache Spark, Druid stores and indexes data and queries it. As a distributed system, Druid uses ZooKeeper for configuration management and consensus management (see [Figure 4-25](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_apache_druid_cluster_consists_of_q)). ZooKeeper plays a critical role in allowing Druid clusters to scale out without management overhead or performance degradation.

[^iiii]: SQL-on-Anything is a query engine that enables users to write SQL queries against files, representational state transfer (REST). APIs, databases, and other sources of data. We will cover this topic in more detail in [Chapter 10](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch10.html#pulsar_sql-id000029).

![This Apache Druid cluster consists of query nodes, coordinator nodes, data storage nodes, and ZooKeeper for configuration management and service discovery.](../img/mapu_0425.png)

*Figure 4-25. This Apache Druid cluster consists of query nodes, coordinator nodes, data storage nodes, and ZooKeeper for configuration management and service discovery.*

# Pulsar Proxy

While Pulsar brokers are the communication mechanism for Pulsar clients, in cases when the brokers are deployed in a private network scenario, we may need a way to expose communication with the outside world. [Figure 4-26](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_pulsar_deployment_on_kubernetesdot_th) shows an example of such a scenario.

![A Pulsar deployment on Kubernetes. The client tries to reach Pulsar over the internet, but the brokers cannot be exposed and Pulsar cannot be reached.](../img/mapu_0426.png)

*Figure 4-26. A Pulsar deployment on Kubernetes. The client tries to reach Pulsar over the internet, but the brokers cannot be exposed and Pulsar cannot be reached.*

A Pulsar proxy is an optional gateway that simplifies the process of exposing brokers to outside traffic. A proxy can be deployed as an additional service and serve the role of taking on internet traffic and routing the messages to the right broker (see [Figure 4-27](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_pulsar_proxy_is_exposing_brokers_o)).

![This Pulsar proxy is exposing brokers on the Kubernetes deployment to the internet so that a client can reach it.](../img/mapu_0427.png)

*Figure 4-27. This Pulsar proxy is exposing brokers on the Kubernetes deployment to the internet so that a client can reach it.*

In many cases, we should have an additional load-balancing layer, called a proxy frontend, to handle the concerns of edge internet traffic (see [Figure 4-28](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#in_this_configurationcomma_the_proxy_ha)).

![In this configuration, the proxy handles all internet traffic. However, it is better suited for routing traffic across brokers.](../img/mapu_0428.png)

*Figure 4-28. In this configuration, the proxy handles all internet traffic. However, it is better suited for routing traffic across brokers.*

Proxy frontends such as HAProxy and NGINX are purpose built for handling internet scale traffic. Using these with a Pulsar proxy as a destination can help ease the load on the proxy (see [Figure 4-29](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_load_balancer_in_front_of_a_pulsar_pr)).

![A load balancer in front of a Pulsar proxy. In this scenario, the load balancer manages communication with clients and the proxy works more as a forwarder.](../img/mapu_0429.png)

*Figure 4-29. A load balancer in front of a Pulsar proxy. In this scenario, the load balancer manages communication with clients and the proxy works more as a forwarder.*

The common thread between ZooKeeper, BookKeeper, and Pulsar is the Java programming language and the Java virtual machine (JVM). Let’s turn our attention to the JVM and the role it plays in Pulsar projects.

# Java Virtual Machine (JVM)

Pulsar brokers, Apache BookKeeper, and Apache ZooKeeper are written in the Java programming language and run on the Java virtual machine (JVM). Earlier in this chapter, we explored a Pulsar broker’s components and noted that a broker is primarily an HTTP server that implements a special TCP protocol (depicted in [Figure 4-30](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#the_pulsar_hierarchy_is_built_on_the_jv)). Nothing within Pulsar screams that it should have been written in Java, so why was Java chosen for Pulsar years ago, and is it still a good choice today?

![The Pulsar hierarchy is built on the JVM with HTTP and a special TCP protocol.](../img/mapu_0430.png)

*Figure 4-30. The Pulsar hierarchy is built on the JVM with HTTP and a special TCP protocol.*

To understand why Yahoo! chose Java, it’s essential to consider the environment in which Pulsar was born. Pulsar’s creation [dates back to 2013](https://oreil.ly/pXKjo), when Yahoo! experienced unparalleled user growth. As Yahoo! grew, it ran into unprecedented challenges in the storage, transmission, and retrieval of data. At the time, the Hadoop ecosystem powered the storage and retrieval systems for most web-scale companies. With Hadoop, many of the tools used to build distributed consensus and configuration management were born and written in Java. Apache Mesos (see [Figure 4-31](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#apache_mesos_is_a_container_orchestrati)) was one of these systems.

![Apache Mesos is a container orchestration engine that is written in Java and utilizes Apache ZooKeeper.](../img/mapu_0431.png)

*Figure 4-31. Apache Mesos is a container orchestration engine that is written in Java and utilizes Apache ZooKeeper.[^iiiii]*

[^iiiii]: Apache Mesos was retired from the Apache Software Foundation in 2020. 



The first and most pragmatic reason to choose Java and the JVM is that many developers know the Java programming language. According to Slashdot, the number of Java developers as of 2018 was 7.3 million. That is a significant percentage of the overall total of 23 million developers. As an open source project, Pulsar can have a lot more success by tapping into a more extensive developer market.

In addition, the Java ecosystem is vast. There are Java libraries for just about everything, and when implementing a platform for all messaging needs, existing libraries can go a long way toward simplifying development.

Finally, Java has an excellent track record of powering essential and scalable solutions in technology. Let’s explore three of them in more depth.

## Netty

Netty is a web server written in Java. It powers the web server infrastructure of companies like Facebook, Apple, Google, and Yahoo! Netty’s two goals are to be portable (run in as many places as possible) and to perform (handle as many concurrent connections as possible). Building a web server requires quality implementations for web protocols, concurrency, and network management. Java has battle-tested implementations for these systems, among others. Netty’s success and use can be attributed to a great developer community, ease of use, and development as a JVM project.

## Apache Spark

Apache Spark is a distributed computing system for in-memory computing. Spark originated at the University of California, Berkeley, and was made freely available through the Apache Software Foundation in 2014. Companies such as Apple, Coinbase, and Capital One use Spark to power their analytics and machine learning. As with Pulsar, Spark developers utilize the JVM for its concurrency primitives and network libraries (the first versions of Spark used Netty for networking), development speed, and reliability. Spark is written in Scala, a programming language that shares the JVM with Java. The interoperability between Scala and Java allows Spark developers to build on the JVM’s rich libraries.

## Apache Lucene

Apache Lucene is an indexing engine that was written in Java and runs on the JVM. Lucene provides the building blocks for search systems such as Elasticsearch, Apache Solr, and CrateDB. Lucene implements the necessary algorithms to index text and perform fuzzy searching over a corpus, and it uses other critical algorithms in search. Search is something we come to expect in the 21st century. Not only can we search the entire web with tools like Google, DuckDuckGo, and Bing, but we can search our email, files on our computers, and even files across our entire presence on the web. Lucene powers the majority of search experiences we encounter on the web.

We covered the positives of the JVM and Java, but there are some negatives as well: notably, the size of JVM applications, the impact of garbage collection on application performance, and the compile times associated with large Java applications. In subsequent chapters, we’ll explore how each of these downsides impacts Pulsar.

# Summary

In this chapter we covered the three primary components that make up a Pulsar cluster: Pulsar brokers, Apache BookKeeper, and Apache ZooKeeper. You learned about the reasoning behind their inclusion in the project and the common thread among them: the Java virtual machine. Now that you know what Pulsar can do, you should be ready to take a closer look at how to use it. In the next few chapters, we’ll discuss the interfaces and tools available in Pulsar, and how to build applications.

