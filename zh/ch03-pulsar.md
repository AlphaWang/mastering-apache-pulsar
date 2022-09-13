# 第 3 章：Pulsar

In [Chapter 2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#event_streams_and_event_brokers), we discussed the motivation for a system like Apache Pulsar: namely, a system that handles both the event stream and pub/sub patterns seamlessly. I provided sufficient evidence for the utility of a system like Pulsar and provided a historical backdrop for asynchronous messaging. We did not, however, cover how Pulsar came into existence. To begin this chapter on the design principles and use cases for Pulsar, it’s worth understanding how exactly the system came to be.

在 [第 2 章](./ch02-event_streams_and_event_brokers.md)中，我们讨论了 Apache Pulsar 这类系统的动机：即一个能无缝处理事件流和发布/订阅模式的系统。 我充分论证了 Pulsar 这类系统的实用性，并讲解了异步消息投递的历史背景。 但是，我们没有介绍 Pulsar 是如何诞生的。 在本章开始介绍 Pulsar 的设计原则和用例之前，有必要了解该系统究竟是如何形成的。

# Pulsar 的起源

In 2013, Yahoo! reported having 800 million active users across its services. At the time, Yahoo! provided services for email, photo storage, news, social media, chat, and fantasy sports, among others. From an infrastructure perspective, Yahoo! felt it needed to address some of its underlying architecture decisions to meet users’ demands and continue to build its world-class services. The messaging architecture used at Yahoo! was thought to be the most important area for improvement. In the company’s service-oriented architecture, the messaging system helped all components scale and provided the low-latency primitives to simplify scalability across all services (see [Figure 3-1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#in_this_representation_of_yahooexclamat)). Following are the most critical aspects for the new messaging platform to meet:

- Flexibility

  Yahoo! worked with queues, publish/subscribe, and streaming, and wanted its messaging platform to handle these use cases seamlessly.

- Reliability

  Yahoo! was accustomed to 99.999% reliability, and the new system had to have the same level of reliability, if not better.

- Performance

  Yahoo! needed low, end-to-end latencies for services like chat and email as well as its ad platform.

2013 年，雅虎已拥有 8 亿活跃用户。当时，雅虎提供的服务包括电子邮件、照片存储、新闻、社交媒体、聊天和梦幻体育等。从基础架构的角度来看，雅虎认为它需要解决一些基础架构决策，以满足用户的需求并继续构建其世界级的服务。 雅虎认为其使用的消息系统架构是最重要的待改进领域。在公司面向服务的架构中，消息系统帮助所有组件进行扩展，并提供低延迟原语以简化所有服务的可扩展性（参见图 3-1)。以下是新消息平台要满足的几个关键因素：

- 灵活性

  雅虎使用队列、发布/订阅和流式处理，并希望其消息平台能够无缝处理这些使用场景。

- 可靠性

  雅虎习惯于 99.999% 的可靠性，新系统必须具有相同水平的可靠性，甚至更好。

- 性能

  雅虎需要为聊天、电子邮件及其广告平台等服务提供很低的端到端延迟。

![In this representation of Yahoo! services, search, news, sports, and email supported by backend infrastructure, including services, databases, and queues.](../img/mapu_0301.png)

*图 3-1. 服务、数据库和队列等后端基础架构支撑了雅虎的搜索、新闻、体育和邮箱服务。*



Yahoo! evaluated existing messaging technologies and determined that none of the open source or off-the-shelf solutions would work for its scale and needs. Yahoo! decided to create a system to meet its needs and designed and built the first version of Pulsar.

雅虎评估了现有的消息技术，发现并没有任何开源或现成的解决方案可以满足其规模和需求。 雅虎决定创建一个新系统来满足其需求，并设计并构建了 Pulsar 的第一个版本。



In 2015, Yahoo! deployed its first Pulsar cluster. Pulsar’s use quickly exploded to replace the company’s existing messaging infrastructure. In 2017, Yahoo! made Pulsar an open source project by donating it to the Apache Software Foundation.

2015 年，雅虎部署了它的第一个 Pulsar 集群。 Pulsar 的使用迅速爆炸式增长，取代了公司原有的消息投递基础设施。 2017 年，雅虎将 Pulsar 捐赠给 Apache 软件基金会，使其成为一个开源项目。



# Pulsar 设计原则

Pulsar was designed from the ground up to be the default messaging platform at Yahoo! As Yahoo! is a large technology company with hundreds of millions of users and numerous popular services, using one platform to meet everyone’s needs was complicated at best. Scalability and usability challenges were only the tip of the iceberg. At the dawn of Pulsar’s design and implementation, many companies were utilizing the public cloud, but cloud adoption was nowhere near what it is today. Creating a system that meets a company’s needs today but doesn’t lock the company into a single development pattern for years to come is a challenge few engineering teams can meet.

Pulsar 从一开始就被设计为雅虎的默认消息平台。由于雅虎是一家拥有数亿用户和众多热门服务的大型科技公司，使用单个平台来满足每个人的需求是复杂的。可扩展性和可用性挑战只是冰山一角。在 Pulsar 设计和实施之初，许多公司都在使用公有云，但云的使用率与今天相去甚远。创建一个既能满足公司今天的需求，又不会在未来几年将公司锁定在单一开发模式的系统，是很少有工程团队能够应对的挑战。



To meet these challenges, the Pulsar designers focused on two essential design principles:

- Modularity
- Performance

为了应对这些挑战，Pulsar 的设计者们专注于两个基本设计原则：

- 模块化
- 性能



Modularity and performance are a rare combination in systems. Like a Thoroughbred made of Legos (see [Figure 3-2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#the_two_design_principles_behind_pulsar)), Pulsar allows for extensions while never compromising on performance. From these two design principles, some of Pulsar’s elegance and foresight come to the surface: namely, its multitenancy, geo-replication, performance, and modularity. We’ll dive into each and show how it relates to Pulsar’s design principles.

模块化和性能是系统中罕见的组合。就像由乐高积木拼成的纯种马（参见图 3-2），Pulsar 允许扩展，同时绝不妥协性能。从这两个设计原则出发，Pulsar 的一些优雅和远见浮出水面：即它的多租户、跨地域复制、性能和模块化。我们将深入研究每一项，并展示其与 Pulsar 设计原则的关系。

![The two design principles behind Pulsar are modularity (represented by the Legos) and performance (represented by the Thoroughbred).](../img/mapu_0302.png)

*图 3-2. Pulsar 背后的两大设计原则是模块化（以乐高积木表示）和性能（以纯种马表示）。*

## 多租户

When I was growing up, I lived in an apartment complex with 27 units in a five-story building. We all shared the utilities, including heat, water, cable, and gas. Trying to ensure a suitable temperature, good water pressure, a reliable cable signal, and adequate gas for all the units was impossible. In the winter, the top floor was 75°F (too hot) and the first floor was 65°F (barely tolerable). In the morning, tenants raced to get to the shower before all the hot water was used and the water pressure was low. If several tenants were watching Monday Night Football, it would be difficult to get a reliable signal.

在我成长的过程中，我曾住过一个五层楼的公寓大楼，有 27 个单元。我们共享公用设施，包括暖气、水、有线电视和燃气。要想确保所有单元都温度合适、水压良好、有线电视信号可靠以及燃气充足是不可能的。在冬天，顶层是 75°F（太热），一楼则是 65°F（勉强可以忍受）。早上，租户们争先恐后地冲到淋浴间，以免热水被用完，以免水压过低。如果有几个租户正在观看周一晚上的足球赛，就很难得到可靠的电视信号。



Our apartment complex was multitenant (see [Figure 3-3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#in_this_apartment_buildingcomma_multipl)) in that each unit contained its own family but all units shared resources. In software, multitenant systems are designed to run workloads for different customers on the same hardware. Pulsar’s flexible subscription model and decoupled architecture enable a high-quality, multitenant experience.

我们的公寓大楼是多租户的（见图 3-3），每个单元都有自己的家庭，但所有单元都共享资源。在软件方面，多租户系统旨在为不同客户在同一硬件上运行工作负载。 Pulsar 灵活的订阅模式和解耦架构可实现高质量的多租户体验。

![In this apartment building, multiple tenants share resources. Without smart management of these resources, one tenant can impact the others. Pulsar’s multitenant architecture enables the use of multiple tenants on fixed resources.](../img/mapu_0303.png)

*图 3-3. 在这座公寓楼里，多个租户共享资源。如果不对这些资源进行智能管理，一个租户可能会影响其他租户。 Pulsar 的多租户架构支持在固定资源上使用多个租户。*



Pulsar handles multitenancy through namespacing and allows tenant scheduling on a specific part of the Pulsar cluster. A *namespace* is simply a logical grouping of topics. Namespaces give structure to Pulsar by providing some organizing fabric to the topics. These two mechanisms put the Pulsar cluster operator in full control of resource allocation and isolation for specific tenants. Returning to the example of my apartment building, Pulsar allows a cluster operator to keep together those tenants who like to keep their unit at the same temperature, and to separate those tenants who take showers around the same time to preserve hot water and water pressure. We’ll explore multitenancy in Pulsar further in [Chapter 4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pulsar_internals).

Pulsar 通过命名空间处理多租户，并允许在 Pulsar 集群的特定部分进行租户调度。 *namespace* 只是主题的逻辑分组。命名空间为主题提供组织结构。这两种机制使得 Pulsar 集群运维人员可以完全控制特定租户的资源分配和隔离。回到我的公寓楼的例子，Pulsar 允许集群运运维人员将那些喜欢将其单元保持在相同温度的租户聚集在一起，并将那些在同一时间洗澡的租户分开以保持热水和水压。我们将在 [第 4 章](./ch04-pulsar_internals.md)中进一步探讨 Pulsar 中的多租户。

## 跨地域复制

Yahoo!, a globally distributed company, had more than 800 million users at its peak. Pulsar’s usage spread throughout Yahoo! and around the globe, and with that expansion came the responsibility of replicating data. Replication is the process of copying data from one node in Pulsar to another. Replication is a core aspect of two important concepts: performance and redundancy. It’s worth taking a moment to consider why these two aspects are important.

雅虎是一家全球化的公司，在鼎盛时期拥有超过 8 亿用户。 Pulsar 的使用遍布整个雅虎以及全球各地，随着这种扩张，复制数据的责任也随之而来。复制是将数据从一个 Pulsar 节点复制到另一个节点的过程。复制是两个重要概念的核心方面：性能和冗余。值得花点时间分析一下为什么这两个方面很重要。



We use computers to perform tasks like word processing, browsing the internet, editing photos, and playing games, to name a few. Our computers use software and a few hardware components to make these experiences possible. As we use our computers, we quickly learn their limitations. We may notice that when we have 20 tabs open in our browser, everything on the computer slows down. We may notice we can’t process a 4K video and stream a movie concurrently. Every computer has a finite set of hardware resources and a quantifiable limit that the hardware can achieve.

我们使用计算机来执行诸如文字处理、浏览互联网、编辑照片和玩游戏等任务。我们的计算机使用软件和硬件组件来实现这些体验。当我们使用计算机时，很快就会了解到它们的局限性。我们可能会注意到，当我们在浏览器中打开 20 个标签时，计算机上的所有内容都会变慢。我们还可能会注意到，我们无法同时处理 4K 视频和流式传输电影。每台计算机都有一套有限的硬件资源，这些硬件能达到的极限是有限的。



When you consider a system like Apache Pulsar, it runs on hardware that is not dissimilar from your personal computer. The hardware has the same memory, processing power, disk space, network speed, and other limitations of a personal computer. When you consider how to get more power out of your computer, you have a few options:

- Make the programs perform better, given your hardware constraints.
- Get a computer with more hardware to meet your requirements.
- Find a clever way to get more hardware.

Apache Pulsar 这类系统的运行硬件与你的个人计算机并没有什么不同。这些硬件具有与个人计算机相同的内存、处理能力、磁盘空间、网络速度和其他限制。当你考虑如何充分利用计算机时，你有以下几种选择：

- 在硬件限制的条件下，让程序性能更好。
- 买一台有更多硬件的计算机来满足你的要求。
- 找到一个巧妙的方法来获得更多的硬件。



The first option is likely already considered and in place. For most of the software we use, like web browsers, email clients, and games, the developers of the software focused on getting the maximum performance from the hardware. Getting a bigger, better computer may be feasible for some budgets, but your needs might quickly outstrip your budget. The clever way to get more hardware is to distribute the needs of our computing system across many computers. This distributed approach to managing hardware requirements is how Pulsar deals with the performance aspect of its responsibilities.

第一个选项可能已经考虑并到位。对于我们使用的大多数软件，如网络浏览器、电子邮件客户端和游戏，软件开发人员将硬件性能最大化。对于某些预算而言，购买更大、更好的计算机可能是可行的，但你的需求可能很快就会超出预算。获得更多硬件的巧妙方法是将我们的计算系统的需求分布在多台计算机上。这种管理硬件需求的分布式方法是 Pulsar 提升其性能的方式。



In [Figure 3-4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#nodes_with_fixed_performance_bandwidths), we have three nodes, labeled N0, N1, and N2. Each is able to handle a specific amount of load from an external process. The smallest load that can be managed is 100 MBps and the largest is 2 GBps. Suppose now that the cost of increasing performance by 100 MBps was exponential. This would mean that as you approach 2 GBps and then 3 GBps, you are paying 10 times more than what you were paying at 100 MBps.

在图 3-4 中，我们有三个节点，分别标记为 N0、N1 和 N2。每个都能够处理来自外部进程的特定数量的负载。可支持的最小负载为 100 MBps，最大负载为 2 GBps。现在假设将性能提高 100 MBps 的成本是指数级的。这意味着当您接近 2 GBps 和 3 GBps 时，您支付的费用是 100 MBps 时支付的费用的 10 倍。

![Nodes with fixed performance bandwidths ranging from 100 MBps to 2 GBps. To scale the system to ingest more data per second we must add another node of the same size.](../img/mapu_0304.png)

*图 3-4. 节点的固定性能带宽从 100 MBps 到 2 GBps 不等。为了扩展系统以便每秒摄取更多数据，我们必须添加另一个相同大小的节点。*



Now consider [Figure 3-5](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#seven_nodescomma_each_with_a_bandwidth). In this figure, we have nodes labeled N0–N6; all have the constraint of 100 MBps, but instead of one node we have seven nodes. If we can effectively split our tasks in a way that can maximize the throughput of each node, we can get the outcomes we want without running into prohibitively expensive costs. So far, all of the drawings of Pulsar have multiple instances of Pulsar, and that is intentional. It is for performance and cost management, but also for redundancy, which we’ll talk about next.

现在我们来看图 3-5。 在这个图中，我们有标记为 N0-N6 的七个节点； 都具有 100 MBps 的带宽约束，但我们有七个节点而不是一个节点。 如果我们能能够有效地拆分我们的任务，使得每个节点的吞吐量最大化，我们就可以获得我们想要的结果，而不会遇到过于昂贵的成本。 到目前为止，所有 Pulsar 的图纸都有多个实例，这是有意为之的。 这是为了性能和成本管理，也是为了冗余，我们接下来会讲到。

![Seven nodes, each with a bandwidth of 100 MBps. In this configuration, all the data in the system is split across the seven nodes.](../img/mapu_0305.png)

*图 3-5. 七个节点，每个节点的带宽为 100 MBps。 在此配置中，系统中的所有数据都分布在七个节点上。*



For a third option, consider [Figure 3-6](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#nodes_with_different_configurations_but). In this figure we have two nodes. N0 contains all the data entries in an encyclopedia, from A to Z. When someone wants to look up a term, the term is retrieved from N0 at a specific speed. Notice that N0 has a fixed disk size of 1 TB. If the encyclopedia requires more than 1 TB of space, we will have to move to another node. The other node, N1, has the same considerations as N0 but more disk space and can take on more capacity. Both N0 and N1 have one critical problem, though: if either of them goes offline, no encyclopedia data can be retrieved. Also, their capacities are fixed, so regardless of what words users are interested in looking up, the nodes contain all the data, useful or not. 

图 3-6 展示了第三种选择。在这个图中，我们有两个节点。 N0 包含百科全书中从 A 到 Z 的所有数据条目。当有人要查找某个词条时，该词条会以特定速度从 N0 中检索出来。请注意，N0 具有 1 TB 的固定磁盘大小。如果百科全书需要超过 1 TB 的空间，我们将不得不移动到另一个节点。另一个节点 N1 与 N0 具有相同的配置，但磁盘空间更大，可以承担更多容量。但是，N0 和 N1 都有一个关键问题：如果其中任何一个离线，则无法检索到百科全书数据。此外，它们的容量是固定的，因此无论用户有兴趣查找哪些单词，节点都包含所有数据，无论有用与否。

![Nodes with different configurations but the same encyclopedia requirements. N1 can store a much larger corpus than N0.](../img/mapu_0306.png)

*图 3-6. 具有不同配置节点，但他们都具有相同的百科全书需求。 N1 可以存储比 N0 大得多的语料库。*



In [Figure 3-7](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#a_coordinator_with_distributed_workload), we see a different model for the encyclopedia. Now, not only are there multiple nodes (N0–N3), but none of them contains the entire encyclopedia; rather, only specific letters. This figure also introduces the concept of a coordinator, which will map a request to the right encyclopedia. In this model we get higher throughput, assuming an even distribution of requests across the encyclopedia. One more step would be to have more than one node share their letters. This way, if an individual node is offline, we can still serve requests for those letters in the encyclopedia.

图 3-7 展示了百科全书的另一个不同模型。现在不仅有多个节点（N0-N3），而且没有一个节点包含整个百科全书；相反，每个节点只包含特定的字母。该图还引入了协调器的概念，它将请求映射到正确的百科全书。在这个模型中，假设请求在整个百科全书中均匀分布，我们可以获得更高的吞吐量。还有一个步骤是让多个节点共享对应的字母。这样，如果单个节点离线，我们仍然可以为百科全书中的这些字母提供请求响应。

![A coordinator with distributed workloads. The coordinator is aware of the responsibility of each node, and as it receives new work, it knows where it should be routed.](../img/mapu_0307.png)

*图 3-7. 具有分布式工作负载的协调器。协调器知道每个节点的责任，当它接收到新的工作时，它知道应该被路由到哪里。*



For Apache Pulsar, both the performance and redundancy considerations of being distributed are fully realized. To build on this further, Pulsar enables distribution across datacenters. In fact, it can be deployed across multiple datacenters by default. This means that, as an application scales across geographies, it can use one Pulsar cluster. Topics are replicated across datacenters, and topic redundancy can be configured to the needs of the applications utilizing the topic. For a topic deployed across hundreds of datacenters around the world, Pulsar manages the complexity of routing data to the right place.

Apache Pulsar 充分实现了分布式带来的的性能和冗余。在此基础上，Pulsar 支持跨数据中心分布。事实上，默认情况下 Pulsar 可以跨多个数据中心部署。这意味着，随着应用程序跨地域扩展，它可以使用单个 Pulsar 集群。主题会跨数据中心复制，主题冗余可以根据使用该主题的应用程序的需要进行配置。对于部署在全球数百个数据中心的主题，Pulsar 负责将数据路由到正确位置，对外隐藏其中的复杂性。



Later in the book, we’ll explore how Pulsar’s replication protocol works and how replication enables smooth global operations for companies like Splunk. For the remainder of this chapter, we’ll focus on how Pulsar components are modular, allowing for a cluster across the globe to appear as one connected cluster (see [Figure 3-8](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#in_geo-replication_in_pulsarcomma_the_p)). This design and implementation detail separates Pulsar from other systems.

本书的后续章节将探讨 Pulsar 的复制协议是如何工作的，以及数据复制如何为 Splunk 等公司实现顺畅的全球运维。本章接下来将关注 Pulsar 组件是如何模块化的，使得全球各地的集群看起来像一个相连的集群（见 图 3-8）。这种设计和实现细节是 Pulsar 独有的。

![In geo-replication in Pulsar, the publisher (producer) sends a message to the brokers, which replicate the messages across geographies as needed.](../img/mapu_0308.png)

*图 3-8. 在 Pulsar 跨地域复制中，发布者（生产者）向 Broker 发送消息，Broker 根据需要跨地理复制消息。*



## 性能

As a messaging system, Pulsar’s primary responsibility is to reliably and quickly consume publishers’ messages and deliver subscribers’ messages. Fulfilling these responsibilities on top of being a reliable message storage mechanism is complicated. From the outset, Yahoo! was concerned about building a system with low latency. In its 2016 post [“Open-Sourcing Pulsar, Pub-Sub Messaging at Scale”](https://oreil.ly/Xu0az), Yahoo! cites average publish latencies of 5 ms as being a strict requirement when building Pulsar. Let’s put this into perspective.

作为一个消息系统，Pulsar 的主要职责是可靠、快速地接收发布者的消息并将消息投递给订阅者。在可靠的消息存储机制之上履行这些职责是复杂的。从一开始，雅虎就关注构建一个低延迟的系统。在其 2016 年的文章[“Open-Sourcing Pulsar, Pub-Sub Messaging at Scale”](https://oreil.ly/Xu0az) 中，雅虎提到在构建 Pulsar 时他们将 5 毫秒的平均发布延迟作为一项严格要求。



One millisecond is one one-thousandth of a second, and the speed of a human eye blink ranges from 100 to 400 milliseconds. Yahoo! required speeds much faster than the blink of an eye just to publish latencies, or the speed at which the message broker receives, saves, and acknowledges the message. Why is this? As I stated earlier, message platforms are often the center of company operations. Publishing to the messaging system is Step 1 among many other steps, but getting safely and quickly to the messaging system is perhaps the most important goal. By ensuring quick publishing times, every other downstream action can begin, and the overall time from when a message was created to when it delivered value is shortened.

一毫秒是千分之一秒，人眼眨眼的速度在 100 到 400 毫秒之间。雅虎用比眨眼速度快得多的速度来要求发布延迟，即 Broker 接收、保存和确认消息的速度。为什么要这样？正如我之前所说，消息平台通常是公司运维的中心。发布消息到消息系统是众多步骤中的第 1 步，但安全快速地发布到消息系统也可能是最重要的目标。通过确保快速发布时间，可以更快地开始其他下游操作，并能缩短从创建消息到投递消息的总时延。



While this book covers all of Pulsar’s features, building blocks, and ecosystem, the reality is that Pulsar’s core functionality is unquestionably fast message delivery. All other features in Pulsar build off of this fundamental truth.

虽然本书涵盖了 Pulsar 的所有功能、构建块和生态系统，但现实情况是，Pulsar 的核心功能无疑是快速的消息投递。 Pulsar 中的所有其他功能都建立在这一基本事实之上。

## 模块化

At its core, Pulsar’s implementation is a distributed log. The distributed log is an excellent primitive for a system like Pulsar because it provides the building blocks for many systems, including databases and file systems. In 2013, Jay Kreps, then a principal staff engineer at LinkedIn, published a blog post titled [“The Log: What every software engineer should know about real-time data’s unifying abstraction”](https://oreil.ly/QF0K0). In this post, Kreps argues that the log provides some key tenants, allowing it to be a building block for real-time systems. Namely, logs are *append only* (meaning you can add to the log but not remove an item from the log) and are indexed based on the order an item was inserted into the log.

Pulsar 的核心是一个分布式日志。分布式日志对于 Pulsar 这类系统来说是一个极好的原语，因为它为许多系统提供了基石，包括数据库和文件系统。 2013 年，时任 LinkedIn 首席工程师的 Jay Kreps 发表了一篇博客文章，标题为[“日志：每个软件工程师都应该了解的实时数据统一抽象”](https://oreil.ly/QF0K0)。在这篇文章中，Kreps 认为日志提供了一些关键租户，使其成为实时系统的基石。也就是说，日志是*仅附加 append only*（意味着你往日志中添加项目，但不能从日志中删除项目），并根据项目插入日志的顺序进行索引。



[Chapter 4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pulsar_internals) describes Pulsar’s implementation of a distributed log in more detail. For now, we’ll focus on how building this core enables other messaging models to work with Pulsar.

[第 4 章](./ch04-pulsar_internals.md) 更详细地描述了 Pulsar 分布式日志的实现。现在，我们将专注于如何构建这个核心，使得 Pulsar 支持其他各种消息传递模型。



In [Chapter 2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#event_streams_and_event_brokers) we talked about event streams. An event stream has a one-to-one relationship with a log. Each event is appended to the stream and ordered by an index (offset). When a consumer publishes an event stream, maintaining the order in which the messages were published is vital. Pulsar’s event-based implementation works well for this use case. For a queue, the order in which the messages are published doesn’t matter. Additionally, the queue consumers don’t care where they are relative to the queue’s beginning or end. In this instance, you can still use the log but relax some of the constraints to model a queue.

在 [第 2 章](./ch02-event_streams_and_event_brokers.md)中，我们讨论了事件流。事件流与日志具有一对一的关系。每个事件都附加到流中并按索引（偏移量）排序。当生产者发布事件流时，维护消息发布的顺序至关重要。 Pulsar 基于事件的实现非常适合这种场景。对于队列，消息发布的顺序无关紧要。此外，队列消费者并不关心他们相对于队列的开始或结束位置。在这种情况下，你仍然可以使用日志来对队列进行建模，只需要放宽一些约束即可。



Alternative pub/sub implementations can be small moderations on top of a distributed log implementation. For example, MQTT (Message Queuing Telemetry Transport) is implemented in Pulsar via the MQTT-On-Pulsar project. Other implementations and protocols can run on top of Pulsar with modifications to the core log, such as the Kafka protocol or the AMQP 1.0 protocol.

其他 pub/sub 实现可以在分布式日志实现之上进行小调整即可。例如，Pulsar 通过 MQTT-On-Pulsar 项目来实现MQTT（Message Queuing Telemetry Transport）。需修改核心日志，其他实现和协议就可以在 Pulsar 之上运行，例如 Kafka 协议和 AMQP 1.0 协议。



# Pulsar 生态

Creating Pulsar and making it an open source project provided the building blocks for sound and flexible messaging. Since then, developers have built powerful tools to couple with Pulsar’s underlying technology. In addition to the three projects highlighted in this section, the Pulsar community is active with thousands of users in Slack channels and messaging boards.

构建出 Pulsar 并使其成为一个开源项目为健全而灵活的消息系统提供了基石。从那时起，开发人员构建了强大的工具来与 Pulsar 的底层技术相结合。除了本节重点介绍的三个项目外，Pulsar 社区在 Slack 频道和消息板上有成千上万用户，非常活跃。



## Pulsar Function

At its core, Pulsar is about performant messaging and storage. We’ve talked at length in this chapter about Pulsar’s flexible design for data storage and scalability. Pulsar Functions answer the question of how to process data stored within Pulsar. They are lightweight compute processes that can consume data from a Pulsar topic, perform some computation, and then publish the results to another Pulsar topic.

Pulsar 的核心是高性能消息投递和存储。我们在本章已经详细讨论了 Pulsar 在数据存储和可扩展性方面的灵活设计。 Pulsar Function 回答了如何处理存储在 Pulsar 中的数据的问题。它们是轻量级计算进程，可以消费来自 Pulsar 主题的数据，执行一些计算，然后将结果发布到另一个 Pulsar 主题。



Pulsar Functions draw inspiration from Functions as a Service implementations such as Google Cloud Functions and Amazon Web Services Lambda Functions. Specifically, Pulsar Functions have a flexible deployment model in which resources can be coupled with Pulsar broker nodes or run as a separate process. Pulsar Functions both receive and output to Pulsar topics (see [Figure 3-9](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#this_pulsar_function_is_receiving_a_top)).

Pulsar Function 从 Google Cloud Functions 和 Amazon Web Services Lambda Functions 等 FaaS（Functions as a Service，功能即服务）系统中汲取灵感。具体来说，Pulsar Functions 具有灵活的部署模型，其资源可以与 Pulsar Broker 节点耦合或作为单独的进程运行。 Pulsar Function 既能接收也能输出到 Pulsar 主题（见图 3-9）。



![This Pulsar function is receiving a topic, performing some processing, and sending it to another topic.](../img/mapu_0309.png)

*图 3-9. Pulsar Function 接收一个主题，执行一些处理，然后将结果发送到另一个主题。*



Pulsar Functions align well with the design principle of modularity. Though Pulsar’s core is written in Java, you can write Pulsar Functions in Java, Python, or Go. The choice to separate Pulsar’s runtime from Pulsar Functions’ runtime reduces the learning curve for programmers who want to learn Pulsar and interact with it. Pulsar Functions are an optional way to process messages in Pulsar. If a user wants to continue using their current stream processing framework, they can do that instead. Pulsar Functions also provide a high-quality stream processing implementation that has a shallow learning curve; if you can write in Java, Python, or Go, you can write semantically correct stream processing without learning a new framework.

Pulsar Function 与模块化的设计原则非常吻合。 虽然 Pulsar 核心是用 Java 编写的，但你可以用 Java、Python 或者 Go 来编写 Pulsar Function。 将 Pulsar 的运行时与 Pulsar Function 的运行时分开的选择，减少了想要学习 Pulsar 并与之交互的程序员的学习曲线。 Pulsar Function 是在 Pulsar 中处理消息的一种可选方式。 如果用户想继续使用他们当前的流处理框架，也是可以的。 Pulsar Function 还提供了一个学习曲线较浅的高质量流处理实现； 如果你可以使用 Java、Python 或 Go 编写代码，则无需学习新框架即可编写语义正确的流处理程序。



## Pulsar IO

Pulsar IO is a connector framework for Pulsar that allows Pulsar topics to become input or output for other processes. To understand Pulsar IO, it’s a bit more instructive to think of an end-to-end example. Suppose you want to create a pipeline that reads in data from your MySQL database row by row and then stores it in an Elasticsearch index. (*Elasticsearch* is an open source search engine technology. An *index* is a named entity in Elasticsearch by which the documents are organized. You can think of them as analogous to a database in relational database parlance.) Pulsar IO can facilitate this entire application with just configuration (see [Figure 3-10](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#a_pulsar_process_pulls_data_from_mysqlc)).

Pulsar IO 是 Pulsar 的连接器框架，允许 Pulsar 主题成为其他进程的输入或输出。要理解 Pulsar IO，用一个端到端的例子来思考会更有启发性。假设你要创建一个管道，从 MySQL 数据库中逐行读取数据，然后将其存储在 Elasticsearch 索引中。 （*Elasticsearch* 是一种开源搜索引擎技术。*index* 是 Elasticsearch 中用于组织文档的命名实体。您可以将它们视为关系数据库术语中的数据库。）Pulsar IO 只需配置即可完成整个应用（见图 3-10）。



![A Pulsar process pulls data from MySQL, and a Pulsar IO process moves data from the Pulsar topic to Elasticsearch.](../img/mapu_0310.png)

*图 3-10. 一个 Pulsar IO 进程从 MySQL 中提取数据，另一个 Pulsar IO 进程将数据从 Pulsar 主题移动到 Elasticsearch。*



Like Pulsar Functions, Pulsar IO provides an isolated and scalable compute process to facilitate the event-driven movement of data through Pulsar topics to destinations. In Pulsar Functions, the interface is a Pulsar topic; in Pulsar IO, the interface can be a Pulsar topic or an external system. Pulsar IO has some philosophical and implementation similarities to Kafka Connect. Like Kafka Connect, Pulsar IO is designed to enable ease of use for everyday use cases with Pulsar. I’ll go into considerably more detail and give examples of using Pulsar IO and building our application in [Chapter 7](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch07.html#pulsar_io-id000027).

与 Pulsar Functions 一样，Pulsar IO 提供了一个隔离的、可扩展的计算过程，以促进数据以事件驱动方式通过 Pulsar 主题移动到目的地。在 Pulsar Functions 中，接口是 Pulsar 主题；在 Pulsar IO 中，接口可以是 Pulsar 主题也可以是外部系统。 Pulsar IO 与 Kafka Connect 在思路和实现上有一些相似之处。与 Kafka Connect 一样，Pulsar IO 旨在为 Pulsar 的日常用例提供易用性。我将在 [第 7 章](./ch07-puslar_io.md) 中更详细地介绍使用 Pulsar IO。



## Pulsar SQL

Pulsar’s decoupled compute and storage architecture allows it to store more data for more extended periods. With all of the data stored in Pulsar, querying data on Pulsar is a natural next step. Pulsar SQL provides a scalable compute runtime that enables SQL queries to be executed against Pulsar topics. Pulsar SQL uses Apache Presto, a SQL-on-Anything engine, to provide the compute resources to query Pulsar topics.

Pulsar 从架构上解耦计算层和存储层，这允许它在更长时间内存储更多数据。 由于所有数据都存储在 Pulsar 中，下一步自然是在 Pulsar 上查询数据。 Pulsar SQL 提供了一个可扩展的计算运行时，使得能够针对 Pulsar 主题执行 SQL 查询。 Pulsar SQL 使用 Apache Presto（一种 SQL-on-Anything 引擎）来提供计算资源查询 Pulsar 主题。



Pulsar SQL has some philosophical similarities to Kafka’s KSQL. Both are designed to allow the querying of topics with SQL syntax. Pulsar SQL is a read-only system designed to query topics and not necessarily create permanent data views. KSQL, on the other hand, is interactive and allows users to create new topics based on the results of SQL queries. You’ll learn more about Pulsar SQL implementation and use cases in [Chapter 10](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch10.html#pulsar_sql-id000029).

Pulsar SQL 与 Kafka 的 KSQL 在设计上有一些相似之处。 两者都旨在通过使用 SQL 语法查询主题。 Pulsar SQL 是一个只读系统，旨在查询主题，而不必创建永久数据视图。 另一方面，KSQL 是交互式的，允许用户根据 SQL 查询的结果创建新主题。 我们将在 [第 10 章](./ch10-pulsar_sql.md) 中了解更多有关 Pulsar SQL 实现和使用案例。



# Pulsar 成功案例

So far in this chapter, we’ve talked at length about how and why Apache Pulsar was developed, the Pulsar ecosystem, and the unique challenges that resulted when Pulsar became an open source system. Can Pulsar address the problems of companies today? Does a platform like Pulsar created in the private cloud of Yahoo! work for companies hosted on the public cloud? These are essential questions for teams that are evaluating Pulsar for their messaging needs. In this section, we’ll cover three companies that adopted Pulsar and have been successful with it. These stories highlight a business problem that engineering teams turned to Pulsar to explain, and they describe how Pulsar helped them solve the problem.

本章到目前为止，已经详细讨论了 Apache Pulsar 的开发方式和原因、Pulsar 生态系统以及 Pulsar 成为开源系统后带来的独特挑战。 Pulsar 能否解决当今公司的问题？ 在雅虎的私有云中创建的 Pulsar 这类平台是否也能适用于托管在公有云上的公司？ 这些问题对于那些正在评估 Pulsar 以满足其消息投递需求的团队来说是必不可少。 在本节中，我们将介绍三家采用 Pulsar 并取得成功的公司。 这些故事突出了工程团队转向 Pulsar 的业务问题，并描述了 Pulsar 如何帮助他们解决问题。



## 雅虎日本

In 2017, Yahoo! JAPAN managed about 70 billion page views per month. At the time, it faced challenges around managing the load on its servers and orchestrating hundreds of services in its service-oriented architecture. Along with the challenges of running its services at scale, Yahoo! JAPAN wished to distribute its entire architecture across countries (geo-replication). Yahoo! JAPAN looked to a messaging system to help with each of these problems. The company investigated Apache Pulsar and Apache Kafka for its workflow needs and reported the results of the investigation in a [blog post published in 2019](https://oreil.ly/8K6Hw).

2017 年，雅虎日本每月管理约 700 亿次页面浏览量。当时，它面临着管理其服务器负载，以及在其面向服务的架构中编排数百个服务的挑战。除了大规模运行服务的挑战外，雅虎日本希望将其整个架构分布在各个国家（跨地域复制）。雅虎日本寻求一个消息系统来帮助解决这些问题。该公司调查了 Apache Pulsar 和 Apache Kafka 的工作流程需求，并在 [2019 年发布的博客文章](https://oreil.ly/8K6Hw) 中报告了调查结果。



The authors of the post detail some of the key differences between Pulsar and Kafka and why they ultimately chose Pulsar for their workloads. The three most important features that influenced their choice were the following:

- Geo-replication
- Reliability
- Flexibility

文章详细介绍了 Pulsar 和 Kafka 之间的一些关键区别，以及他们最终选择 Pulsar 来处理工作负载的原因。影响他们选择的三个最重要的特性如下：

- 跨地域复制
- 可靠性
- 灵活性



While Pulsar and Kafka scored similarly on reliability, Pulsar took a commanding lead in terms of geo-replication and flexibility. At the time of their investigation, little had been published on cross-datacenter deployments of Kafka. Meanwhile, the Pulsar story around geo-replication was well known in the community (as stated in this chapter’s opening). Ultimately, Yahoo! JAPAN chose Pulsar and has used it for many years to power its services. Pulsar provides the engine for its service-oriented architecture and removes many of the burdens that come with geo-replication.

虽然 Pulsar 和 Kafka 在可靠性方面得分相似，但 Pulsar 在跨地域复制和灵活性方面处于领先地位。在他们进行调查时，关于 Kafka 跨数据中心部署的报道很少。同时，Pulsar 关于跨地域复制的故事在社区中广为人知（如本章开头所述）。最终，雅虎日志选择了 Pulsar，并且多年来一直使用它来支持其服务。Pulsar 为其面向服务的架构提供了引擎，并消除了跨地域复制带来的许多负担。



## Splunk

Splunk is a corporation that makes the collection, aggregation, and searching of logs and other telemetry data easy (see [Figure 3-11](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#applications_and_databases_forward_thei)). Hundreds of enterprise technology companies use Splunk to collect logs from their applications, instrument their applications, and troubleshoot their infrastructure and applications. In 2019, Splunk acquired Streamlio, the first managed offering of Apache Pulsar. In the [press release announcing the acquisition](https://oreil.ly/o98lq), Splunk notes that Apache Pulsar is a unique technology and that it will be transformative for the company. It’s not hard for a company like Splunk to imagine how technology like Pulsar is used in its products. In a 2020 talk titled [“How Splunk Mission Control Leverages Various Pulsar Subscription Types”](https://oreil.ly/PPQ77), Pranav Dharma, then a senior software engineer at Splunk, covers how Splunk uses Pulsar’s flexible subscription model to power its center of operations. The flexible subscription allows the company to provide a range of message processing guarantees based on application needs. We’ll talk about subscriptions in more detail in [Chapter 6](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch06.html#producers).

Splunk 是一家旨在更容易地收集、聚合和搜索日志和其他遥测数据的公司（见图 3-11)。数以百计的企业技术公司使用 Splunk 从他们的应用程序中收集日志，检测他们的应用程序，并对他们的基础设施和应用程序进行故障排除。 2019 年，Splunk 收购了 Streamlio，这是 Apache Pulsar 的第一个托管产品。在 [宣布收购的新闻稿](https://oreil.ly/o98lq) 中，Splunk 指出 Apache Pulsar 是一项独特的技术，它将为公司带来变革。对于像 Splunk 这样的公司来说，不难想象 Pulsar 这样的技术是如何在其产品中使用的。在 2020 年题为 [“Splunk Mission Control 如何利用各种 Pulsar 订阅类型”](https://oreil.ly/PPQ77) 的演讲中，Splunk 高级软件工程师 Pranav Dharma 介绍了 Splunk 如何使用 Pulsar 的灵活订阅模型为其运营中心提供动力。灵活的订阅允许公司根据应用需求提供一系列消息处理保证。我们将在 [第 6 章](./ch06-producers.md) 中更详细地讨论订阅。

![Applications and databases forward their logs and metrics to Splunk, and Splunk indexes them and makes them searchable.](../img/mapu_0311.png)

*图 3-11. 应用程序和数据库将它们的日志和指标转发到 Splunk，Splunk 进行索引并使其可搜索。*



In another 2020 talk titled [“Why Splunk Chose Pulsar”](https://oreil.ly/LA1DA), Karthik Ramasamy, distinguished engineer at Splunk and founder of Streamlio, details the wins that Splunk gets from Pulsar geo-replication, low-latency message transmission, and scalable storage via Apache BookKeeper. Pulsar was a significant investment for Splunk, but by all accounts, it was a worthwhile one. Splunk leverages Pulsar’s core performance to make quick decisions and its flexible subscription model to provide a single platform to handle all messaging needs.

在 2020 年另一场题为 [“为什么 Splunk 选择 Pulsar”](https://oreil.ly/LA1DA) 的演讲中，Splunk 的杰出工程师和 Streamlio 的创始人 Karthik Ramasamy 详细介绍了 Splunk 从 Pulsar 跨地域复制、低延迟消息传输、Apache BookKeeper 可扩展存储中得到的好处。 Pulsar 是 Splunk 的一项重大投资，但从各方面来看，它都是值得的。 Splunk 利用 Pulsar 的核心性能做出快速决策，并利用其灵活的订阅模式提供单一平台来处理所有消息投递需求。



## Iterable

Iterable is a customer engagement platform designed to make customer lifecycle marketing, recommendation systems, and cross-channel engagement easy. To scale its operations across thousands of customers, Iterable needed a messaging platform that could be the foundation of its software interactions. Initially, Iterable used RabbitMQ, but the company ran into the system’s limitations and turned to other systems to solve its messaging problems. In an article titled [“How Apache Pulsar Is Helping Iterable Scale Its Customer Engagement Platform”](https://oreil.ly/7bg34), author Greg Methvin lays out the problems Iterable looked to solve with a new messaging platform. The three key features he and his team looked for were:

Iterable 是一个客户参与平台，旨在简化客户生命周期营销、推荐系统和跨渠道参与。为了在数千名客户中扩展业务，Iterable 需要一个可以作为其软件交互基础的消息平台。最初，Iterable 使用 RabbitMQ，但该公司遇到了该系统的限制，并转向其他系统来解决其消息传递问题。在一篇题为 [“Apache Pulsar 如何帮助 Iterable 扩展其客户参与平台”](https://oreil.ly/7bg34) 的文章中，作者 Greg Methvin 列出了 Iterable 希望通过新的消息传递平台解决的问题。他和他的团队寻找的三个关键特性是：



- Scalability

  Iterable needed a system that would scale up to the demand of its users.

- Reliability

  Iterable needed a system that could reliably store its messaging data.

- Flexibility

  Iterable needed a system that would handle all of its messaging needs.

- 可扩展性

  Iterable 需要一个可以扩展以满足其用户需求的系统。

- 可靠性

  Iterable 需要一个能够可靠地存储其消息数据的系统。

- 灵活性

  Iterable 需要一个能够处理其所有消息投递需求的系统。



Iterable evaluated several messaging platforms, including Apache Kafka, Amazon’s Simple Queue Service (SQS), and Kinesis. In the evaluation, Pulsar was the only system that provided the required semantics and scalability. Iterable used its messaging platform for both queuing and streaming. While Kinesis and Kafka provided some facilities for accomplishing this, they fell short of Pulsar’s elegance and general-purpose mechanism. Additionally, Pulsar’s decoupled architecture provided the flexibility Iterable needed to scale topics independently, as well as the proper semantics in terms of topics.

Iterable 评估了多个消息平台，包括 Apache Kafka、亚马逊的简单队列服务 (SQS) 和 Kinesis。在评估中，Pulsar 是唯一提供所需语义和可扩展性的系统。 Iterable 将其消息传递平台用于队列和流式处理。虽然 Kinesis 和 Kafka 提供了一些工具来实现这一点，但它们没有达到 Pulsar 的优雅和通用机制。此外，Pulsar 的解耦架构提供了独立扩展主题所需的灵活性 Iterable，以及主题方面的适当语义。



By choosing Pulsar as the event backbone of its architecture, Iterable has been able to scale and meet new and growing customer demands.

通过选择 Pulsar 作为其架构的事件基石，Iterable 已经能够扩展来满足新的不断增长的客户需求。



# 总结

In this chapter, we focused exclusively on the use cases for Apache Pulsar, and specifically on some large companies that have used (and continue to use) Pulsar as a cornerstone technology. You learned that Pulsar is especially suitable for the following:

- Low-latency messaging requirements
- Geo-replication
- Problems that require queueing and event streams

在本章中，我们专门讨论了 Apache Pulsar 的使用案例，特别是一些已经使用（并继续使用）Pulsar 作为基础技术的大公司。 我们了解到 Pulsar 特别适用于以下情况：

- 低延迟的消息投递要求
- 跨地域复制
- 需要队列和事件流的问题



We covered the need for streaming technology in [Chapter 1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch01.html#the_value_of_real-time_messaging), we discussed the publish/subscribe method in [Chapter 2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#event_streams_and_event_brokers), and now we have some sufficient motivation for the uniqueness of Pulsar and are prepared to unpack its pieces in [Chapter 4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pulsar_internals).

我们在 [第 1 章](./ch01-the_value_of_real-time_messaging.md) 中讨论了对流技术的需求，在[第 2 章](./ch02-event_streams_and_event_brokers.md)讨论了发布/订阅模式，现在我们对 Pulsar 的独特性有了一些充分的动机，并且准备好在 [第 4 章](./ch04-pulsar_internals.md) 对其抽丝剥茧。