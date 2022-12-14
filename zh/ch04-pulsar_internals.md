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



## 消息存储

Every messaging system has some implementation for storing messages temporarily. A message broker’s value is in the reliable transport of messages, after all. How we implement that storage will have some downstream consequences on how the data can be used later. For systems like Pulsar, Kafka, and Pravega, message durability is paramount. BookKeeper’s model of ledger storage is the perfect abstraction for storing an event stream (see [Figure 4-15](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_topic_has_sequential_data_written)).

每个消息系统都会临时临时存储消息。 毕竟，消息 Broker 的价值在于消息的可靠传输。 我们如何实现该存储会影响以后如何使用数据。 对于 Pulsar、Kafka 和 Pravega 等系统，消息的持久性至关重要。 BookKeeper 的 Ledger 存储模型是存储事件流的完美抽象（见 图 4-15）。



![This topic has sequential data written to BookKeeper ledgers.](../img/mapu_0415.png)

*图 4-15. 主题将数据顺序写入写入 BookKeeper Ledger。*



Following are some of the properties that make BookKeeper a good solution for event stream data:

- Append-only logging
- Highly durable
- Easily distributed

以下属性使得 BookKeeper 成为事件流数据的良好解决方案：

- 仅追加日志
- 高度持久性
- 易于分布式



## 对象/Blob 存储

Object stores allow the storage of arbitrarily large objects for future retrieval. Systems like Amazon S3, Google Cloud Storage, and Azure Blob Storage are popular because they offer a simple API to store and retrieve items from the cloud. In cloud systems, object storage is used for storing images, arbitrary files on behalf of users, and large data lakes. Implementing an object store requires elasticity, or the ability to add new nodes to the cluster without disrupting ongoing operations. It also requires fault tolerance; if nodes in the cluster fail, there should be a reliable backup in the cluster somewhere. In addition, it requires the ability to store and retrieve objects of all kinds. Apache BookKeeper can perform all these tasks and can perform them well. BlobIt is an object store built on top of BookKeeper. It allows for the storage of arbitrarily large objects, and all of the storage is managed with BookKeeper. A user can send a CSV file to BlobIt and the file will be stored on BookKeeper as depicted in [Figure 4-16](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#you_can_store_a_csv_file_on_bookkeeper).

对象存储允许存储任意大的对象以便将来进行检索。 Amazon S3、Google Cloud Storage 和 Azure Blob Storage 等系统很受欢迎，因为它们提供了简单的 API 来存储和检索云中的对象。在云系统中，对象存储用于存储图像、代表用户的任意文件和大型数据湖。实现对象存储需要弹性，在不中断正在进行的操作的情况下向集群添加新节点的能力。它还需要容错性；如果集群中的节点发生故障，集群中的某个地方应该有可靠的备份。此外，它需要存储和检索各种对象的能力。 Apache BookKeeper 可以执行所有这些任务，并且可以很好地执行。 BlobIt 是一个建立在 BookKeeper 之上的对象存储。它允许存储任意大的对象，并且所有的存储都由 BookKeeper 管理。用户可以将 CSV 文件发送到 BlobIt，该文件将存储在 BookKeeper 中，如图 4-16 所示。



![You can store a CSV file on BookKeeper for use as a general-purpose object store.](../img/mapu_0416.png)

*图 4-16. 可将 BookKeeper 作为通用对象存储，用来存储 CSV 文件。*



While BookKeeper can store arbitrarily large data, the complexity in using it as an object store is in managing the movement of the data to and from bytes. BlobIt relies on the distributed and fault-tolerant nature of BookKeeper, and adds value by making an Amazon S3–compliant API.

虽然 BookKeeper 可以存储任意大的数据，但将其用作对象存储的复杂性在于管理数据与字节之间的移动。 BlobIt 依赖于 BookKeeper 的分布式和容错特性，并通过符合 Amazon S3 的 API 来增加价值。



## Pravega

Pravega is a distributed messaging system that has a lot of similarities to Pulsar. Developed at Dell, Pravega builds on the concept of a stream as the fundamental building block for storage. Pravega uses BookKeeper in a similar way to Pulsar (see [Figure 4-17](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pravega_architecturedot_similar_to_apac)): storing all topic and cursor data. Like Pulsar, BookKeeper enables Pravega to scale storage and message throughput independently, and it provides durability for and fault tolerance within a Pravega cluster.

Pravega 是一个分布式消息系统，与 Pulsar 有很多相似之处。 Pravega 由戴尔开发，基于流的概念作为存储的基本构件。 Pravega 使用 BookKeeper 的方式与 Pulsar 类似（见图 4-17）：存储所有主题和游标数据。与 Pulsar 一样，BookKeeper 使 Pravega 能够独立扩展存储和消息吞吐量，并为 Pravega 集群提供持久性和容错性。



![Pravega architecture. Similar to Apache Pulsar, Pravega uses BookKeeper for long-term storage, uses ZooKeeper for distributed coordination, and has some responsibilities that are owned by the Pravega servers.](../img/mapu_0417.png)

*图 4-17. Pravega 架构图。与 Apache Pulsar 类似，Pravega 使用 BookKeeper 进行长期存储，使用 ZooKeeper 进行分布式协调，并且 Pravega 服务器负责一些职责。*



An additional interesting tidbit about Pravega is that its use cases extend beyond just event streaming data (an area that Pulsar is focused on). Pravega is also suitable for streaming video data and large files. As mentioned previously*,* you can store any data on BookKeeper; the challenges lie in how that data is presented and how end users interact with it.

关于 Pravega 的另一个有趣的花絮是，它的使用场景不仅限于事件流数据（Pulsar 关注的领域）。 Pravega 也适用于流媒体视频数据和大文件。如前所述，可以在 BookKeeper 上存储任何数据；挑战在于如何呈现数据以及最终用户如何与之交互。



## Majordodo

Majordodo is a resource manager that handles the scheduling of bespoke workloads on ephemeral clusters. Majordodo tracks the resources used in a cluster, the available resources in a cluster, and other metadata about jobs running in a cluster (see [Figure 4-18](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_majordodo_clusterdot_bookkeeper_manag)). [Majordodo](https://oreil.ly/kh1T4) utilizes BookKeeper ledgers to track the starting, running, and completion of jobs on the cluster. Since BookKeeper provides low read and write latencies, scheduling workloads is a novel but worthy use. Majordodo is developed and maintained by Diennea, a technology company that helps build scalable digital brand solutions.

Majordodo 是一个资源管理器，用于处理临时集群上的定制工作负载的调度。 Majordodo 跟踪集群中使用的资源、集群中的可用资源以及有关集群中运行的作业的其他元数据（见图 4-18）。 [Majordodo](https://oreil.ly/kh1T4) 利用 BookKeeper Ledger 来跟踪集群上作业的启动、运行和完成。由于 BookKeeper 提供低读写延迟，调度工作负载是一种新颖但值得的使用场景。 Majordodo 由 Diennea 开发和维护，该公司是一家帮助构建可扩展数字品牌解决方案的技术公司 。



![A Majordodo cluster. BookKeeper manages data storage for each node so that any node in the cluster can pick up work or distribute existing work.](../img/mapu_0418.png)

*图 4-18. Majordodo 集群示意图。 BookKeeper 管理每个节点的数据存储，这样集群中的任何节点都可以获取工作或分发现有的工作。*



In the preceding sections, we spent a lot of time talking about the importance and use cases for BookKeeper. ZooKeeper works in conjunction with BookKeeper and plays a different but equally important role in Pulsar and in the wider software ecosystem. ZooKeeper is discussed in the next section.

在前面的部分中，我们花了很多时间讨论 BookKeeper 的重要性和使用案例。 ZooKeeper 与 BookKeeper 一起工作，在 Pulsar 和更广泛的软件生态系统中扮演着不同但同样重要的角色。 下一节将讨论 ZooKeeper。



# Apache ZooKeeper

Apache ZooKeeper was developed at Yahoo! in the late 2000s, and the Apache Software Foundation made it an open source platform in 2011. ZooKeeper implements the system described in the 2006 paper “The Chubby Lock Service for Loosely-Coupled Distributed Systems” by Mike Burrows,[^iii] a distinguished engineer at Google. The paper explains why Google needed Chubby to manage its sprawling internal systems and provides some high-level descriptions of its implementation.

Apache ZooKeeper 雅虎于 2000 年代末开发的，并与 2011 年被 Apache 软件基金会开源。ZooKeeper 实现了谷歌工程师 Mike Burrows 在 2006 年的论文“The Chubby Lock Service for Loosely-Coupled Distributed Systems”[^iii] 中描述的系统。 该论文解释了为什么 Google 需要 Chubby 来管理其庞大的内部系统，并提供了一些关于其实现的高层描述。

[^iii]: Mike Burrows, “Chubby lock service for loosely-coupled distributed systems,” *OSDI ’06: Proceedings of the 7th Symposium on Operating Systems Design and Implementation* (November 2006): 24.



Chubby provides tools for distributed configuration management, service discovery, and a two-phase commit implementation. Chubby is a proprietary service used within Google, and the paper provides a peek at how Google handled a standard set of distributed system problems. With some light shed on how to approach these problems, Yahoo! implemented Apache ZooKeeper.

Chubby 提供了用于实现分布式配置管理、服务发现和两阶段提交的工具。 Chubby 是 Google 内部使用的专有服务，文章简要介绍了 Google 如何处理一系列标准的分布式系统问题。 有了一些关于如何解决这些问题启发，雅虎实现了 Apache ZooKeeper。



ZooKeeper provides an open source implementation suitable for coordinating distributed systems. ZooKeeper’s primary requirements are:

- Performance
- Fault tolerance
- Reliability

ZooKeeper 提供了一个适合协调分布式系统的开源实现。 ZooKeeper 的主要需求是：

- 性能
- 容错性
- 可靠性



By meeting these requirements, ZooKeeper is suitable for implementing several distributed system algorithms, including Paxos and Raft. Another standard implementation on top of ZooKeeper is the two-phase commit protocol. The two-phase commit ensures atomicity, or that all nodes have a shared understanding of the system’s current state in a distributed system. The two-phase commit is illustrated in [Figure 4-19](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#in_this_two-phase_commitcomma_a_new_cha).

通过满足这些要求，ZooKeeper 适用于实现多种分布式系统算法，包括 Paxos 和 Raft。 ZooKeeper 之上的另一个标准实现是两阶段提交协议。两阶段提交确保原子性，或者所有节点对分布式系统中的系统当前状态有共同的理解。两阶段提交如图 4-19 所示。



![In this two-phase commit, a new change is accepted by the leader and immediately sent to followers as part of a single transaction.](../img/mapu_0419.png)

*图 4-19. 在两阶段提交中，领导者接受新的更改并立即将其作为单个事务的一部分发送给追随者。*



In Pulsar, ZooKeeper manages the BookKeeper configuration and distributed consensus, and stores metadata about Pulsar topics and configuration. It plays an integral role in all Pulsar operations, and replacing it would be difficult. It’s worth diving into a few examples of use cases for ZooKeeper to understand its importance to Pulsar.

在 Pulsar 中，ZooKeeper 管理 BookKeeper 的配置和分布式共识，并存储有关 Pulsar 主题和配置的元数据。它在所有 Pulsar 操作中都扮演着不可或缺的角色，要替换它是很困难的。值得深入研究 ZooKeeper 的一些使用场景案例，以了解它对 Pulsar 的重要性。



## 命名服务

One common way systems integrate with Apache ZooKeeper is as a naming service. A naming service maps network resources to their respective addresses. In a system with many nodes, keeping track of their identity and their place in the network can be tricky. Apache Mesos uses ZooKeeper for this purpose (among others). In a Mesos cluster, ZooKeeper stores every node, their status, and a leader or follower. If nodes need to coordinate, ZooKeeper can be used as a lookup. ZooKeeper serves this purpose in Apache Pulsar as well, as depicted in [Figure 4-6](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#broker_one_is_not_the_leader_for_topic) earlier in this chapter.

与 Apache ZooKeeper 进行集成的系统的一种常见方式是作为命名服务。 命名服务将网络资源映射到它们各自的地址。 在一个多节点系统中，跟踪节点身份以及节点在网络中的位置可能很棘手。 Apache Mesos 使用 ZooKeeper 来实现这个目的。 在 Mesos 集群中，ZooKeeper 存储每个节点、它们的状态以及领导者或追随者信息。 如果节点需要协调，可以使用 ZooKeeper 进行查找。 ZooKeeper 在 Apache Pulsar 中也有此用途，如本章 图 4-6 所示。



## 配置管理

Apache Pulsar has about 150 configuration values that are tunable by Pulsar operators. Each value changes the underlying behavior of Pulsar, ZooKeeper, or BookKeeper. Some of those configurations impact the publishing and consumption of messages in the Pulsar cluster. Pulsar brokers store their configuration in ZooKeeper because a reliable and highly available place to retrieve and store those configurations is paramount.

Apache Pulsar 有大约 150 个配置值，可由 Pulsar 运维人员进行调整。 每个值都会改变 Pulsar、ZooKeeper 或 BookKeeper 的底层行为。 其中一些配置会影响 Pulsar 集群中消息的发布和消费。 Pulsar Broker 将他们的配置存储在 ZooKeeper 中，因为有一个可靠的、高度可用的地方来检索和存储这些配置是至关重要的。



In some ways, ZooKeeper is a safe, distributed storage engine. As [Figure 4-20](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_zookeeper_cluster_can_store_multiple) shows, ZooKeeper can keep track of named keys and values.

从某些方面说，ZooKeeper 是一个安全的分布式存储引擎。 正如图 4-20所示，ZooKeeper 可以跟踪命名的键和值。



![A ZooKeeper cluster can store multiple configuration values.](../img/mapu_0420.png)

*图 4-20. ZooKeeper 集群可以存储多个配置值。*



## 领导者选举

Leader election is the process of choosing a leader for a specific set of responsibilities in a distributed system. In Apache Pulsar, a broker is the leader of a topic (or one or more partitions in a partitioned topic). If that broker goes offline, a new broker is elected the leader of that same topic or partition(s). Building on both the naming service use case and the configuration use case, ZooKeeper can provide a reliable building block for implementing leader election. It keeps track of leaders, knows where they are in the cluster, and can be called on to implement new leaders in the future, as depicted in [Figure 4-21](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#in_this_leader-follower_modelcomma_the).

领导者选举是在分布式系统中为一组特定职责选择一个领导者的过程。 在 Apache Pulsar 中，Broker 是主题（或分区主题中的一个或多个分区）的领导者。 如果该 Broker 下线，则新的 Broker 将被选为该主题或分区的领导者。 基于命名服务使用场景和配置管理使用场景，ZooKeeper 可以为实现领导者选举提供可靠构件。 它跟踪领导者，知道他们在集群中的位置，并且可以在未来被调用以实现新的领导者，如图 4-21 所示。



![In this leader-follower model, the values are tracked across ZooKeeper nodes.](../img/mapu_0421.png)

*图 4-21. 在领导者-追随者模型中，配置值跨 ZooKeeper 节点进行跟踪。*



## 通知系统

The final ZooKeeper use case that we’ll cover is that of a notification system. In [Chapter 1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch01.html#the_value_of_real-time_messaging), you learned how notifications can help patients in a hospital receive better care. The most important aspects of a notification system are the timely delivery of notifications and the guaranteed delivery of notifications. If you miss a notification about engagement on a tweet you sent late last night, that isn’t a world-stopping event. However, if you miss a notification to renew your driver’s license, you may be arrested the next time you are pulled over. We’ve discussed how ZooKeeper serves as a high-quality naming service. The same qualities that make ZooKeeper a good naming service make it an excellent notification system. Namely, we can ensure that the system state is shared by all parties, and that if a party doesn’t have that notification, we can quickly determine it using ZooKeeper. [Figure 4-22](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#the_commit_protocol_used_in_zookeeper_i) provides a high-level view of this concept.

我们要介绍的最后一个 ZooKeeper 使用场景是通知系统。在 [第 1 章](./ch01-the_value_of_real-time_messaging.md)中，你了解了通知是如何帮助医院的患者获得更好的护理。通知系统中最重要的是通知的及时投递和通知的保证投递。如果你错过了关于你昨晚深夜发送的推文的评论通知，那并不是一个震惊世界的事件。但是，如果你错过了更新驾驶执照的通知，那么在下次被拦下时久可能会被捕。我们已经讨论了 ZooKeeper 如何充当高质量的命名服务，相同的品质使其成为出色的通知系统。也就是说，我们可以确保系统状态由各方共享，并且如果一方没有该通知，我们可以使用 ZooKeeper 快速确定它。 图 4-22提供了这个概念的高层视图。



![The commit protocol used in ZooKeeper is useful as a notification protocol.](../img/mapu_0422.png)

*图 4-22. ZooKeeper 中使用的提交协议可用作通知协议。*



## Apache Kafka

Apache Kafka is a distributed messaging system suitable for event streaming. It has broad adoption because of its thoughtful API design and scalability characteristics. Developed at LinkedIn and made freely available in 2014, Kafka provides the building blocks for event management and real-time systems at companies around the world. As of version 2.5, Kafka utilizes Apache ZooKeeper for configuration management and leader election. ZooKeeper plays a critical role in fault tolerance and message delivery in a Kafka cluster. [Figure 4-23](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_apache_kafka_cluster_has_three_bro) depicts a Kafka cluster with Apache ZooKeeper.

Apache Kafka 是一个适用于事件流场景的分布式消息系统。由于其周到的 API 设计和可扩展性特性，它得到了广泛应用。 Kafka 由 LinkedIn 开发，并于 2014 年免费提供，为世界各地的公司提供事件管理和实时系统的构件。从 2.5 版开始，Kafka 使用 Apache ZooKeeper 进行配置管理和领导者选举。 ZooKeeper 在 Kafka 集群中的容错和消息传递中起着至关重要的作用。 图 4-23 描绘了一个带有 Apache ZooKeeper 的 Kafka 集群。



![This Apache Kafka cluster has three brokers, and ZooKeeper for coordination and leader election.](../img/mapu_0423.png)

*图 4-23. Apache Kafka 集群示例：有三个 Broker，以及用于协调和领导选举的 ZooKeeper。*



Interestingly, the Kafka project removed the requirement of ZooKeeper in a Kafka cluster as of version 2.8 and replaced the ZooKeeper responsibilities with a Raft consensus implementation within the cluster itself. [Figure 4-24](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_kafka_cluster_has_three_nodes_with) depicts this change.

有趣的是，从 2.8 版本开始，Kafka 项目删除了 Kafka 集群对 ZooKeeper 的要求，并用集群本身内的 Raft 共识实现替换了 ZooKeeper 职责。 图 4-24描述了这一变化。



![This Kafka cluster has three nodes with their own consensus algorithm, known as KRaft (Kafka Raft).](../img/mapu_0424.png)

*图 4-24. 该 Kafka 集群有三个节点，有自己的共识算法，称为 KRaft（Kafka Raft）。*



## Apache Druid

Apache Druid is a real-time analytics database originally developed by Metamarkets in 2011 and made freely available through the Apache Software Foundation in 2015. Druid powers the analytics suite of companies like Alibaba, Airbnb, and [Booking.com](http://booking.com/). Unlike SQL-on-Anything engines[^iiii] such as Presto and Apache Spark, Druid stores and indexes data and queries it. As a distributed system, Druid uses ZooKeeper for configuration management and consensus management (see [Figure 4-25](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_apache_druid_cluster_consists_of_q)). ZooKeeper plays a critical role in allowing Druid clusters to scale out without management overhead or performance degradation.

Apache Druid 是一个实时分析数据库，最初由 Metamarkets 于 2011 年开发，并于 2015 年通过 Apache 软件基金会免费提供。Druid 为阿里巴巴、Airbnb 和 Booking.com 等公司的分析套件提供动力。与 Presto 和 Apache Spark 等 SQL-on-Anything 引擎[^iiii] 不同，Druid 存储和索引数据并对其进行查询。作为分布式系统，Druid 使用 ZooKeeper 进行配置管理和共识管理（见图 4-25）。 ZooKeeper 在允许 Druid 集群中发挥着重要作用，保障横向扩展而无需管理开销以及无需损失性能。

[^iiii]: SQL-on-Anything 是一个查询引擎，使用户能够编写针对文件、表征状态传输 (REST)、 API、数据库和其他数据源的 SQL 查询。我们将在 [第 10 章](./ch10-pulsar_sql.md) 中更详细地介绍这个主题。



![This Apache Druid cluster consists of query nodes, coordinator nodes, data storage nodes, and ZooKeeper for configuration management and service discovery.](../img/mapu_0425.png)

*图 4-25. Apache Druid 集群由查询节点、协调节点、数据存储节点和用于配置管理和服务发现的 ZooKeeper 组成。* 



# Pulsar Proxy

While Pulsar brokers are the communication mechanism for Pulsar clients, in cases when the brokers are deployed in a private network scenario, we may need a way to expose communication with the outside world. [Figure 4-26](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_pulsar_deployment_on_kubernetesdot_th) shows an example of such a scenario.

Pulsar Broker 是 Pulsar 客户端与集群进行通信的机制，但如果 Broker 部署在私有网络中，我们可能需要用其他方式来进行通信。 图 4-26 展示了这种场景的一个示例。

![A Pulsar deployment on Kubernetes. The client tries to reach Pulsar over the internet, but the brokers cannot be exposed and Pulsar cannot be reached.](../img/mapu_0426.png)

*图 4-26. 部署在 Kubernetes 上的 Pulsar。客户端尝试通过 Internet 访问 Pulsar，但无法访问 Broker，亦即无法访问 Pulsar。* 



A Pulsar proxy is an optional gateway that simplifies the process of exposing brokers to outside traffic. A proxy can be deployed as an additional service and serve the role of taking on internet traffic and routing the messages to the right broker (see [Figure 4-27](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#this_pulsar_proxy_is_exposing_brokers_o)).

Pulsar Proxy 是一个可选的网关，它简化了将 Broker 暴露给外部流量的过程。Proxy 可以作为一项附加的服务部署，并起到接收互联网流量并将消息路由到正确 Broker 的作用（见图 4-27)。

![This Pulsar proxy is exposing brokers on the Kubernetes deployment to the internet so that a client can reach it.](../img/mapu_0427.png)

*图 4-27. Pulsar Proxy 将部署在 Kubernetes 上的 Broker 暴露给互联网，这样客户端可以访问到 Broker。*



In many cases, we should have an additional load-balancing layer, called a proxy frontend, to handle the concerns of edge internet traffic (see [Figure 4-28](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#in_this_configurationcomma_the_proxy_ha)).

在许多情况下，还应该有一个额外的负载平衡层，称为代理前端，来处理边缘互联网流量的问题（见图 4-28)。

![In this configuration, the proxy handles all internet traffic. However, it is better suited for routing traffic across brokers.](../img/mapu_0428.png)

*图 4-28. 在此配置中，Proxy 处理所有互联网流量。但是，它更适合跨 Broker 进行流量路由。*



Proxy frontends such as HAProxy and NGINX are purpose built for handling internet scale traffic. Using these with a Pulsar proxy as a destination can help ease the load on the proxy (see [Figure 4-29](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#a_load_balancer_in_front_of_a_pulsar_pr)).

HAProxy 和 NGINX 等代理前端专为处理互联网规模的流量而设计。将它们与 Pulsar Proxy 一起使用可以帮助减轻 Proxy 上的负载（见图 4-29）。

![A load balancer in front of a Pulsar proxy. In this scenario, the load balancer manages communication with clients and the proxy works more as a forwarder.](../img/mapu_0429.png)

*图 4-29. 负载均衡器部署在 Pulsar Proxy 前面。在这种情况下，负载均衡器管理与客户端的通信，Proxy 更多充当转发器角色。*



The common thread between ZooKeeper, BookKeeper, and Pulsar is the Java programming language and the Java virtual machine (JVM). Let’s turn our attention to the JVM and the role it plays in Pulsar projects.

ZooKeeper、BookKeeper 和 Pulsar 之间的共同点是 Java 编程语言和 Java 虚拟机 (JVM)。让我们把注意力转向 JVM 及其在 Pulsar 项目中所扮演的角色。



# Java 虚拟机 (JVM)

Pulsar brokers, Apache BookKeeper, and Apache ZooKeeper are written in the Java programming language and run on the Java virtual machine (JVM). Earlier in this chapter, we explored a Pulsar broker’s components and noted that a broker is primarily an HTTP server that implements a special TCP protocol (depicted in [Figure 4-30](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#the_pulsar_hierarchy_is_built_on_the_jv)). Nothing within Pulsar screams that it should have been written in Java, so why was Java chosen for Pulsar years ago, and is it still a good choice today?

Pulsar Broker、Apache BookKeeper 和 Apache ZooKeeper 都是用 Java 编程语言编写的，并在 Java 虚拟机 (JVM) 上运行。 在本章前面，我们探讨了 Pulsar Broker 的组件，并注意到 Broker 主要是一个实现了一个特殊的 TCP 协议的 HTTP 服务器，如图 4-30 所示。 Pulsar 内部没有任何东西表明它应该用 Java 编写，那么为什么多年前 Pulsar 选择了 Java，现在它仍然是一个好选择吗？



![The Pulsar hierarchy is built on the JVM with HTTP and a special TCP protocol.](../img/mapu_0430.png)

*图 4-30. Pulsar 层次结构建立在 JVM 上，带有 HTTP 和特殊的 TCP 协议。*



To understand why Yahoo! chose Java, it’s essential to consider the environment in which Pulsar was born. Pulsar’s creation [dates back to 2013](https://oreil.ly/pXKjo), when Yahoo! experienced unparalleled user growth. As Yahoo! grew, it ran into unprecedented challenges in the storage, transmission, and retrieval of data. At the time, the Hadoop ecosystem powered the storage and retrieval systems for most web-scale companies. With Hadoop, many of the tools used to build distributed consensus and configuration management were born and written in Java. Apache Mesos (see [Figure 4-31](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#apache_mesos_is_a_container_orchestrati)) was one of these systems.

要了解雅虎为什么选择 Java，就必须考虑 Pulsar 诞生的环境。 Pulsar 的创作 [追溯到 2013 年](https://oreil.ly/pXKjo)，当时雅虎经历了前所未有的用户增长。随着雅虎的发展，它在数据存储、传输和检索方面遇到了前所未有的挑战。当时，Hadoop 生态系统为大多数大规模公司的存储和检索系统提供支持。借助 Hadoop，许多用于构建分布式共识和配置管理的工具诞生了，并且都是用 Java 编写的。 Apache Mesos（见图 4-31）就是这种系统之一。

![Apache Mesos is a container orchestration engine that is written in Java and utilizes Apache ZooKeeper.](../img/mapu_0431.png)

*图 4-31。 Apache Mesos 是一个用 Java 编写的并利用 Apache ZooKeeper 的容器编排引擎。[^iiiiii]*

[^iiiii]: Apache Mesos 于 2020 年从 Apache 软件基金会退役。 



The first and most pragmatic reason to choose Java and the JVM is that many developers know the Java programming language. According to Slashdot, the number of Java developers as of 2018 was 7.3 million. That is a significant percentage of the overall total of 23 million developers. As an open source project, Pulsar can have a lot more success by tapping into a more extensive developer market.

选择 Java 和 JVM 的首要也是最务实的原因是，许多开发人员都了解 Java 编程语言。根据 Slashdot 的数据，截至 2018 年，Java 开发人员的数量为 730 万。这在 2300 万开发者总数中占很大比例。作为一个开源项目，Pulsar 可以通过进入更广泛的开发者市场取得更大的成功。



In addition, the Java ecosystem is vast. There are Java libraries for just about everything, and when implementing a platform for all messaging needs, existing libraries can go a long way toward simplifying development.

此外，Java 生态系统是巨大的。几乎所有东西都有 Java 库，当实现一个满足所有消息传递需求的平台时，现有的库可以大大简化开发。



Finally, Java has an excellent track record of powering essential and scalable solutions in technology. Let’s explore three of them in more depth.

最后，Java 在为技术中的基本和可扩展解决方案提供支持方面有着出色的记录。让我们更深入地探讨其中的三个。



## Netty

Netty is a web server written in Java. It powers the web server infrastructure of companies like Facebook, Apple, Google, and Yahoo! Netty’s two goals are to be portable (run in as many places as possible) and to perform (handle as many concurrent connections as possible). Building a web server requires quality implementations for web protocols, concurrency, and network management. Java has battle-tested implementations for these systems, among others. Netty’s success and use can be attributed to a great developer community, ease of use, and development as a JVM project.

Netty 是一个用 Java 编写的 Web 服务器。 它为 Facebook、Apple、Google 和雅虎等公司的 Web 服务器基础架构提供支持。 Netty 的两个目标是可移植（在尽可能多的地方运行）和执行（处理尽可能多的并发连接）。 构建 Web 服务器需要高质量的 Web 协议、并发和网络管理实现。 Java 已经为这些系统等提供了久经考验的实现。 Netty 的成功和使用可以归功于优秀的开发者社区、易用性以及作为 JVM 项目的开发。



## Apache Spark

Apache Spark is a distributed computing system for in-memory computing. Spark originated at the University of California, Berkeley, and was made freely available through the Apache Software Foundation in 2014. Companies such as Apple, Coinbase, and Capital One use Spark to power their analytics and machine learning. As with Pulsar, Spark developers utilize the JVM for its concurrency primitives and network libraries (the first versions of Spark used Netty for networking), development speed, and reliability. Spark is written in Scala, a programming language that shares the JVM with Java. The interoperability between Scala and Java allows Spark developers to build on the JVM’s rich libraries.

Apache Spark 是一个用于内存计算的分布式计算系统。 Spark 起源于加州大学伯克利分校，并于 2014 年通过 Apache 软件基金会免费提供。Apple、Coinbase 和 Capital One 等公司使用 Spark 为其分析和机器学习提供支持。 与 Pulsar 一样，Spark 开发人员将 JVM 用于其并发原语和网络库（Spark 的第一个版本使用 Netty 进行联网）、开发速度和可靠性。 Spark 是用 Scala 编写的，这是一种与 Java 共享 JVM 的编程语言。 Scala 和 Java 之间的互操作性允许 Spark 开发人员在 JVM 的丰富库上进行构建。



## Apache Lucene

Apache Lucene is an indexing engine that was written in Java and runs on the JVM. Lucene provides the building blocks for search systems such as Elasticsearch, Apache Solr, and CrateDB. Lucene implements the necessary algorithms to index text and perform fuzzy searching over a corpus, and it uses other critical algorithms in search. Search is something we come to expect in the 21st century. Not only can we search the entire web with tools like Google, DuckDuckGo, and Bing, but we can search our email, files on our computers, and even files across our entire presence on the web. Lucene powers the majority of search experiences we encounter on the web.

Apache Lucene 是一个用 Java 编写并在 JVM 上运行的索引引擎。 Lucene 为 Elasticsearch、Apache Solr 和 CrateDB 等搜索系统提供构件。 Lucene 实现了必要的算法来索引文本并在语料库上执行模糊搜索，并且它在搜索中使用了其他关键算法。 搜索是我们在 21 世纪所期待的。 我们不仅可以使用 Google、DuckDuckGo 和 Bing 等工具搜索整个网络，还可以搜索我们的电子邮件、计算机上的文件，甚至是整个网络上的文件。 Lucene 为我们在网络上遇到的大多数搜索体验提供支持。



We covered the positives of the JVM and Java, but there are some negatives as well: notably, the size of JVM applications, the impact of garbage collection on application performance, and the compile times associated with large Java applications. In subsequent chapters, we’ll explore how each of these downsides impacts Pulsar.

我们讨论了 JVM 和 Java 的优点，但它们也有一些缺点：特别是 JVM 应用程序的大小、垃圾收集对应用程序性能的影响，以及大型 Java 应用程序的编译时间。 在随后的章节中，我们将探讨这些缺点如何影响 Pulsar。



# 总结

In this chapter we covered the three primary components that make up a Pulsar cluster: Pulsar brokers, Apache BookKeeper, and Apache ZooKeeper. You learned about the reasoning behind their inclusion in the project and the common thread among them: the Java virtual machine. Now that you know what Pulsar can do, you should be ready to take a closer look at how to use it. In the next few chapters, we’ll discuss the interfaces and tools available in Pulsar, and how to build applications.

在本章中，我们介绍了构成 Pulsar 集群的三个主要组件：Pulsar Broker、Apache BookKeeper 和 Apache ZooKeeper。 你了解了 Pulsar 包含这几个组件的原因以及它们之间的共同点：Java 虚拟机。 既然你知道 Pulsar 可以做什么，那么你应该准备好仔细研究如何使用它。 在接下来的几章中，我们将讨论 Pulsar 中可用的接口和工具，以及如何构建应用程序。



