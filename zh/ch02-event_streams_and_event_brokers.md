# 第 2 章：事件流与事件 Broker

Event streams and event brokers are at the heart of every real-time system. An *event stream* is an endless series of events. Let’s revisit the banking example in [Chapter 1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch01.html#the_value_of_real-time_messaging). A borrower’s financial transactions can be considered an event stream. Each time a borrower uses their credit card, applies for a new line of credit, or deposits a check, those actions or events are appended to their event stream. Since the event stream is infinite, the bank can use it to return to any point in the borrower’s past. If the bank wanted to know what a borrower’s bank account looked like on a specific day in history, it could reconstruct that from an event stream. The event stream is a powerful concept, and when equipped with this data, it can empower organizations and developers to make life-changing experiences.

事件流和事件 Broker 是每个实时系统的核心。 *事件流*是一系列无穷无尽的事件。让我们回顾一下 [第 1 章](./ch01-the_value_of_real-time_messaging.md) 中的银行业务示例。借款人的金融交易可以被视为一个事件流。每次借款人使用信用卡、申请新的信用额度或存入支票时，这些操作或事件都会附加到他们的事件流中。由于事件流是无限的，银行可以使用它查看借款人过去任何时间点。如果银行想知道借款人的银行账户在历史上某一天的样子，可以从事件流中重建出来。事件流是一个强大的概念，组织和开发人员能够使用事件流创造出改变生活的体验。



Event brokers are the technology platforms that store event streams and interact with clients that read data from or write data to event streams. Apache Pulsar is an event broker at heart. However, calling Pulsar *only* an event broker would minimize its scope and impact. To fully understand what makes Pulsar unique, it is beneficial to dive into some of the strengths and weaknesses of event brokers and their approaches to implementing event streams. This chapter will walk through some historical context and motivate a discussion around the need for Apache Pulsar.

事件 Broker 是存储事件流的技术平台，并与从事件流读取数据或向事件流写入数据的客户端进行交互。Apache Pulsar 本质上是一个事件 Broker 。但是，把 Pulsar *仅仅* 看做是事件 Broker 就太局限了。为了充分了解 Pulsar 的独特之处，就得深入了解事件 Broker 的一些优势和劣势，及其实现事件流的方法。本章将介绍一些历史背景，并围绕 Apache Pulsar 的需求展开讨论。



# 发布/订阅

Developers across disciplines in software engineering commonly use the publish/subscribe pattern. At its core, this pattern decouples software systems and smooths the user experience of asynchronous programming. Popular messaging technologies like Apache Pulsar, Apache Kafka, RabbitMQ, NATS, and ActiveMQ all utilize the publish/subscribe pattern in their protocols. It’s worth jumping into this pattern’s history to understand its significance and build on why Pulsar is unique.

软件工程中各学科的开发人员经常使用发布/订阅模式。这种模式的核心是对软件系统进行解耦，使异步编程的用户体验更加流畅。Apache Pulsar、Apache Kafka、RabbitMQ、NATS 和 ActiveMQ 等流行的消息系统技术都在其协议中使用了发布/订阅模式。有必要深入了解这种模式的历史，以理解其重要性并引申出 Pulsar 之所以独特的原因。



In 1987, Kenneth Birman and Thomas Joseph published a paper titled “Exploiting virtual synchrony in distributed systems” in the *ACM SIGOPS Operating Systems Review*.[^i] Their paper describes an early implementation of a large-scale messaging platform built on the publish/subscribe pattern. In their paper, the authors make a convincing case around the publish/subscribe pattern’s value. Specifically, they claim that systems implemented this way *feel* synchronous, even though they are inherently asynchronous. To illustrate this point more clearly, let’s dive into the publish/subscribe pattern with some examples.

1987 年，Kenneth Birman 和 Thomas Joseph 在 *ACM SIGOPS Operating Systems Review* 上发表了一篇题为“Exploiting virtual synchrony in Distributed Systems”的论文。[^i] 这篇论文描述了一个建立在发布/订阅模式上的大规模消息平台的早期实现。作者围绕发布/订阅模式的价值提出了一个令人信服的案例。具体来说，他们声称以这种方式实现的系统尽管本质上是异步的，但*感觉*上却是同步的。为了更清楚地说明这一点，让我们通过一些示例深入了解发布/订阅模式。

[^i]: Kenneth Birman and Thomas Joseph, “Exploiting virtual synchrony in distributed systems,” *ACM SIGOPS Operating Systems Review 21*, no. 5 (November 1987): 123–138.



The idea of a subscription is commonplace in the 21st century. I have subscriptions to news services, entertainment, food delivery, loyalty programs, and many others. [Figure 2-1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#the_publishsolidussubscribe_pattern_dec) is a simple illustration of a pub/sub pattern. I *subscribe* to goods for services from a retailer or entertainer, and they deliver me goods or services based on an agreement. The news services I subscribe to are the best way to illustrate a publish/subscribe pattern. I subscribe to a news service, and when a news source publishes a new article, I expect to receive it on my phone. In this example, you can consider my news service provider to be an event broker, the news publication to be a producer, and me to be a consumer. There are a few features in this relationship that are worth pointing out.

在 21 世纪，订阅的想法是很普遍的。我订阅了新闻服务、娱乐、食品递送、用户忠诚度计划等。[图2-1] 是pub/sub模式的简单说明。我*订阅*来自零售商或艺人的服务商品，他们根据协议为我提供商品或服务。新闻服务订阅是阐明发布/订阅模式的最佳方式。我订阅了一个新闻服务，当一个新闻源发布了一篇新文章，我期望在我的手机上收到它。在这个例子中，你可以把我的新闻提供者看作是一个事件经纪人，把新闻发布者看做是一个生产者，而把我看做是一个消费者。我要特别指出这种关系中的几个特点。



![The publish/subscribe pattern decouples software systems and smooths the user experience of asynchronous programming.](../img/mapu_0201.png)

*图2-1. 发布/订阅模式对软件系统进行解耦，使异步编程的用户体验更加流畅。*



First, there is no coupling between the news publication (publisher) and the subscriber (consumer). The news publication does not need to know that I’m a subscriber; they focus on writing articles and sending them to the news service. Similarly, I don’t need to know anything about the mechanisms of the news publication; the news service provides a reliable mechanism for publication and consumption. From a consumer perspective, promptly getting news on my phone feels magical. I can control how many messages I receive per day and what times I prefer to receive them. For the news publication, they can focus on producing quality news. Delivering the news to the right customers at the right time is managed by the news service.

首先，新闻发布者（发布者）和订阅者（消费者）之间没有耦合。新闻发布者不需要知道我是一个订阅者；他们专注于撰写文章并将它们发送到新闻服务。同样，我不需要知道任何关于新闻发布者的机制；新闻服务为新闻发布和消费提供了一个可靠的机制。从消费者的角度来看，及时在手机上获得新闻感觉很神奇。我可以控制每天收到多少条消息，以及我喜欢在什么时候收到这些消息。对于新闻发布者来说，他们可以专注于生产高质量的新闻。在正确的时间将新闻传递给正确的客户，则由新闻服务机构管理。



The aha moment in the virtual synchrony paper was that the publish/subscribe pattern makes asynchronous workflows feel synchronous. [Figure 2-2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#in_this_publishsolidussubscribe_topolog) depicts this model. Examining the interactions with my news service from all angles, it does feel synchronous. I don’t feel like I have to ask and wait for a relationship with my news publication; it just shows up when I need it. The publisher publishes their stories to the news service, and their stories are in users’ hands.

令人惊叹的是发布/订阅模式使异步工作流感觉起来是同步的。 [图 2-2] 描述了这个模型。从各个角度审视与我的新闻服务的各种交互，感觉确实是同步的。我不需要询问和等待与我的新闻发布者的关系；它只是在我需要的时候出现。发布者将他们的故事发布到新闻服务，然后他们的故事就会出现在用户手中。



![In this publish/subscribe topology, there are multiple producers and consumers.](../img/mapu_0202.png)

*图 2-2. 在这个发布/订阅拓扑中，有多个生产者和消费者。*



The event stream implements a publish/subscribe pattern, but it has one critical distinction: the event broker must retain the same order for every subscriber. This distinction may not seem like much at first blush, but it enables a whole new way of using the publish/subscribe pattern. Consider our example of the news service. When a new customer signs up for the service, they will receive news articles in the future but likely will not receive all past messages onto their device on sign-up. Most messaging systems guarantee the delivery of a published message to a current subscriber and purposefully release messages that are already delivered. For an event stream, the event broker retains the entire history of data. When a new consumer subscribes to the event stream, they choose where they want to start consuming from (including the beginning of time).

事件流实现了发布/订阅模式，但它有一个关键区别：事件 Broker 必须为每个订阅者保持相同的顺序。乍一看似乎这种区别不大，但它使发布/订阅模式有了全新的使用方式。以我们的新闻服务为例，当新客户注册该服务时，他们将来会收到新闻文章，但在注册时可能不会在他们的设备上收到所有过去的消息。大多数消息传递系统保证将已发布消息传递给当前订阅者，并有目的地释放已传递的消息。对于事件流，事件 Broker 会保留整个数据历史记录。当一个新的消费者订阅事件流时，他们会选择他们想要从哪里开始消费。



# 队列

A queue is a different approach to a publish/subscribe pattern. In the publish/subscribe pattern discussed in the previous section, every subscriber to a topic receives a published message. In the queue model, only one subscriber will receive a message published to a topic. The queue model tackles a specific kind of publish/subscribe problem where each message in the queue is waiting on work to be completed, and the subscribers perform that work. Consider the process of being invited to a party (see [Figure 2-3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#a_generic_invitation_is_shown_on_the_le)). An invitation to anyone who has access to them is analogous to a queue (left) and an invitation to a specific person is analogous to an event stream (right).

队列不同于发布/订阅模式。在上一节讨论的发布/订阅模式中，每个主题的订阅者都会收到一条发布的消息。在队列模型中，只有一个订阅者会收到发布到主题的消息。队列模型解决了一种特定类型的发布/订阅问题，队列中的每条消息都在等待工作完成，而订阅者执行该工作。例如被邀请参加聚会的过程（参见 [图 2-3]）。对任何有权限的人的邀请类似于队列（左），对特定人员的邀请类似于事件流（右）。



The queue model is more straightforward than the event stream model and works for a larger class of applications in which a client receives a work unit, then publishes the work unit to the messaging system, and a downstream consumer completes the work. Email, unsubscribing, deleting records, orchestration of events, and indexing are examples of this class of applications.

队列模型比事件流模型更直接，适用于更大类的应用程序，其中客户端接收工作单元，然后将工作单元发布到消息传递系统，下游消费者完成工作。电子邮件、取消订阅、删除记录、事件编排和索引都是此类应用。



![A generic invitation is shown on the left and an invitation for a specific invitee (Kelsey) is shown on the right.](../img/mapu_0203.png)

*图 2-3. 左侧为通用邀请，右侧为特定受邀者 (Kelsey) 的邀请。*



Some messaging systems are purpose-built for the queue model. This class of messaging system is called a *work queue.* Work queues are designed for managing the workloads of programmatic processes. They have three purposes: 1) keep track of all the work to be done (a queue); 2) allow the appropriate worker to perform work; and 3) report back their completion (or lack thereof) of work. Beanstalkd is a widely used messaging system that is a work queue. Beanstalkd’s design is purposefully simple and doesn’t require the user to configure it. Let’s walk through an example in some more depth to get a better handle on Beanstalkd and the queue model.

有些消息系统是专门为队列模型构建的。此类消息传递系统称为*工作队列。*工作队列旨在管理程序化进程的工作负载。它们有三个目的：1）跟踪所有要完成的工作（队列）； 2) 允许合适的工作者执行工作； 3) 报告他们完成的（或未完成的）工作。 Beanstalkd 是一个广泛使用的工作队列消息系统。 Beanstalkd 的设计很简单，不需要用户进行配置。让我们通过一个更深入的例子来更好地掌握 Beanstalkd 和队列模型。



Beanstalkd organizes work into logical lists called tubes. You can think of a tube as a queue; it is an ordered list of work to be completed, and it has a name. Inside a tube is a job; since Beanstalkd is a work queue, most of the language and concepts are aligned with *work.* Beanstalkd has *publishers*, or programmatic clients that are asking for work to be complete, and it has *subscribers*, or programmatic clients that are picking up work and then marking it as complete. Here is a simple Beanstalkd client publishing a new job to the “worka” tube:

Beanstalkd 将工作组织成逻辑列表，称为 `tube`。你可以把 `tube` 想象成一个队列；它是待完成工作的有序列表，并且有一个名称。由于 Beanstalkd 是一个工作队列，因此大多数语言和概念都与 *work 保持一致。* Beanstalkd 有 *publisher*，即要求工作完成的程序化客户端；还有 *subscriber* ，即接收工作并将其标记为完成的程序化客户端。如下是一个简单的 Beanstalkd 客户端向“worka” tube 发布新工作：

```
// A python program that connects to a local instance of Beanstalkd and creates a
// job
import beanstalkc
beanstalk = beanstalkc.Connection(host='localhost', port=14711)
beanstalk.use('worka') 
beanstalk.put('my job 123')
```



Now that we’ve published work, we can connect to the same “worka” tube and complete the work:

现在我们已经发布了工作，我们可以连接到同一个“worka” tube 并完成工作：

```
import beanstalkc

beanstalk = beanstalkc.Connection(host='localhost', port=14711)
beanstalk.use('worka')
job = beanstalk.reserve() // reserves the job in the tube
job.body // prints "my job 123"
job.delete() // Deletes the job from beanstalkd, marking it was complete
```

In this model, the Beanstalkd server is responsible for keeping track of which jobs go to which tubes, but the consumer manages most of the complexity in the system. The consumer is responsible for the following:

- Reserving the work
- Marking the work as complete
- Resubmitting the work if it fails
- Maintaining a connection with Beanstalkd

在此模型中，Beanstalkd 服务器负责跟踪哪些作业进入哪些 tube，但消费者管理着系统中的大部分复杂性。 消费者负责以下事项：

- 保留工作
- 将工作标记为完成
- 如果失败，重新提交工作
- 保持与 Beanstalkd 的连接



In Beanstalkd, typically one job goes to one worker (subscriber).[^ii] When we zoom in on the purpose of the work queue, the decision to have a one-to-one relationship with a job and a consumer (worker) is reasonable. However, there are some implicit side effects of this queue model that we should explore in some more detail to understand the differences.

在 Beanstalkd 中，通常一项工作交给一个工作者（订阅者）。[^ii] 当我们放大工作队列的目的时，决定对工作和消费者（工作者）建立一对一的关系是合理的。 然而，这个队列模型有一些隐含的副作用，我们应该更详细地探索以了解这些差异。

[^ii]: The Beanstalkd protocol explicitly states that one subscriber should consume a job. However, there are some hacks you can implement to ensure that multiple subscribers consume a job. For example, by simply never deleting a job, you can allow every subscriber to see that job once it’s released.  Beanstalkd 协议明确规定一个订阅者应该消费一个作业。 但是你可以实施一些技巧来让多个订阅者使用一个作业。 例如，通过简单地从不删除作业，您可以允许每个订阅者在发布后看到该作业。



First, in the queue model (and specifically in the Beanstalkd API), we assume that when a job is complete there is no need for it anymore. In fact, in Beanstalkd, the worker should explicitly delete the job when it’s no longer being worked on. Second, there is no order preservation for the jobs. This means there is no guarantee that a job arriving at Beanstalkd would be processed in any specific or consistent order. For some applications it may be necessary to allow multiple subscribers to pick up the same job, and it may also be advantageous to have a way to follow how jobs arrived in the queue from a historical perspective.

首先，在队列模型中（特别是在 Beanstalkd API 中），我们假设当工作完成后就不再需要它。 事实上，在 Beanstalkd 中，工作者应该在不再工作时显式删除工作。 其次，工作不保序。 这意味着无法保证到达 Beanstalkd 的工作会以任何特定或一致的顺序进行处理。 对于某些应用程序，可能需要允许多个订阅者处理同一个工作，并且需要有一种方法从历史的角度跟踪作业如何到达队列。



# 故障模式

Messaging systems can fail. They can fail to deliver messages to subscribers, they can fail to accept publishers’ messages, and they can lose messages in transit. Contingent on the system’s design and use, each failure can have varying degrees of severity. If we think back on the email examples earlier in this chapter, failure to deliver email can have a varying degree of severity. If you fail to receive your favorite email newsletter in your inbox on a given day, that is not the end of the world. You may spend your time on other, more fulfilling pursuits in the absence of the newsletter. But if a failure occurs in an ecommerce platform’s payment pipeline, and the ecommerce platform uses email messages to create *virtual synchrony*, it can prevent a user from receiving their products in the best case and bankrupt the business in the worst case. It’s essential to build a messaging platform that is resistant to failures.

消息系统可能会失败。他们可能无法将消息投递给订阅者，也可能无法接受发布者的消息，还可能会在传输过程中丢失消息。取决于系统的设计和使用情况，每个故障可能带来不同程度的严重性。如果我们回想本章前面的电子邮件示例，发送电子邮件失败的严重程度可能不同。如果您未能在某一天收到您最喜欢的电子邮件时事通讯，那并不是世界末日。没有时事通讯时，您可能会将时间花在其他更充实的追求上。但是，如果电子商务平台的支付管道发生故障，并且电子商务平台使用电子邮件消息来创建*虚拟同步*，在最好情况下它可能会让用户无法收到他们的产品，在最坏的情况下则可能使业务破产。构建一个能够抵抗故障的消息平台至关重要。



Managing the three failures of message acceptance, message delivery, and message storage requires thoughtful design and wise implementations. We’ll discuss how Pulsar tackles these issues in the next chapter.

管理消息接受、消息传递和消息存储这三种失败需要周到的设计和明智的实施。我们将在下一章讨论 Pulsar 如何解决这些问题。



# 推送与轮询

When a producer publishes a new message to a queue or event stream, the way that message propagates to consumers can vary. The two mechanisms for pushing that message to consumers are pushing and polling.

生产者向队列或事件流发布新消息，而消息传播到消费者的方式可能会有所不同。将消息推送给消费者的两种机制是推送和轮询。



In the *push* model, the event broker pushes messages to a consumer with some predefined configuration. For example, the broker may have a fixed number of messages per period that it sends to a consumer, or it may have a maximum number of messages queued before it pushes them to the consumer. A majority of messaging systems today use a push mechanism because brokers are eager to move messages off their hands.

在 *push* 模型中，事件 Broker 通过一些预定义的配置将消息推送给消费者。例如， Broker 可能在每个周期向消费者发送固定数量的消息，或者当队列消息达到最大数量后将消息推送给消费者。今天的大多数消息传递系统都使用推送机制，因为 Broker 急于将消息从他们手中移走。



In an event system, queued messages have some value, but processing the messages is the system’s end goal. By eagerly pushing messages to available consumers, the event broker can rid itself of the responsibility for the message. However, as discussed in [“Failure Modes”](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#failure_modes), an event broker may try to push a consumer message and the consumer may be unavailable. This failure mode necessitates the event broker to retry or, in the queue case, move the message onto another consumer.

在事件系统中，排队的消息具有一定的价值，但处理消息是系统的最终目标。通过急切地将消息推送给可用的消费者，事件 Broker 可以摆脱对消息的责任。但是，正如 “故障模式” 中所讨论的，事件 Broker 可能会尝试推送消费者消息但消费者可能不可用。这种故障模式需要事件 Broker 进行重试，或者在队列情况下将消息转移到另一个消费者。



An alternative to the push model is the *poll* model. The poll model requires the consumer to ask the event broker for new messages. The consumer may ask for new messages after a configured time interval or may ask based on a downstream event. The advantage of this model is that the consumer is always ready to receive messages when it asks. The disadvantage is that the consumer may not receive messages on time or receive them at all.

推送模型的替代方案是 *poll* 模型。轮询模型要求消费者向事件 Broker 索取新消息。消费者可能会在配置的时间间隔请求新消息，也可能会根据下游事件进行请求。这种模型的优点是消费者在请求时总是已经准备好接收消息。缺点是消费者可能无法按时收到消息或根本不会收到消息。



# 对 Pulsar 的需求

So far in this chapter we’ve talked about early systems developed to tackle messaging, and we’ve touched on systems like RabbitMQ, ActiveMQ, and Apache Kafka. These systems require a nontrivial number of resources to develop and a large community to remain viable in a developer market. Why do we need another one? Apache Pulsar addresses three problems that are not addressed by other event broker technologies:

- Unification of streaming and queues
- Modularity
- Performance

到目前为止，在本章中，我们已经讨论了为处理消息投递而开发的早期系统，也提到了 RabbitMQ、ActiveMQ 和 Apache Kafka 等系统。 这些系统需要大量的资源来开发，并且需要庞大的社区才能在开发者市场中保持活力。 那为什么我们还需要另一个呢？ Apache Pulsar 解决了其他事件 Broker 技术未解决的三个问题：

- 流和队列的统一
- 模块化
- 性能



## 统一

The event stream requires an ordered sequence for messages. That ordered sequence enables the rich applications described in [Chapter 1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch01.html#the_value_of_real-time_messaging) and is used in many of the applications you use every day. However, an event stream has specific semantics requiring consumers to manage how they process the stream’s events. What if an application doesn’t require the use of an ordered sequence? What if each client needed to get the next available event and was not concerned about its place in the stream?

事件流需要消息有序。消息有序才能支持 [第 1 章](./ch01-the_value_of_real-time_messaging.md) 中描述的丰富应用程序，并用于你日常使用的许多应用程序。但是，事件流具有特定的语义，要求消费者管理他们如何处理流中的事件。如果应用程序不需要使用有序序列怎么办？如果每个客户端都需要获取下一个可用事件并且不关心它在流中的位置怎么办？



Pulsar allows topics to be either a queue or an event stream, as depicted in [Figure 2-4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#in_a_shared_subscriptioncomma_every_sub). This flexibility means a Pulsar cluster can provide the platform for all the interactions discussed in this chapter.

Pulsar 允许主题既可以是队列也可以是事件流，如图 2-4 所示。这种灵活性意味着 Pulsar 集群可以为本章讨论的所有交互提供平台。



![In a shared subscription, every subscriber gets every message generated by the producer.](../img/mapu_0204.png)

*图 2-4. 在共享订阅中，每个订阅者都会收到生产者生成的每条消息。Figure 2-4. In a shared subscription, every subscriber gets every message generated by the producer.*



## 模块化

We talked about the differences between the queue and event stream models of the publish/subscribe model. While these models differ enough to warrant different architectures, application developers are likely to need both models for building robust software. It’s not uncommon for software development teams to use one system intentionally designed for event streams and another for queueing. While this “best tool for the job” approach can be wise, it does have some downsides. One downside is the operational burden of managing two systems. Each system is unique in its maintenance schedule and procedure, best practices, and operations paradigm. An additional downside is that programmers have to familiarize themselves with multiple paradigms and APIs to write applications.

我们讨论了发布/订阅模型的事件流模型和队列之间的区别。虽然这些模型的差异足以需要不同的架构，但应用程序开发人员可能同时需要这两种模型来构建强大的软件。软件开发团队使用一个专门为事件流设计的系统以及另一个用于队列的系统并不少见。虽然这种“最好的工作工具”方法可能是明智的，但它确实有一些缺点。一个缺点是管理两个系统的运维负担，每个系统的维护计划和程序、最佳实践和操作范式方面都是独一无二的。另一个缺点是程序员必须熟悉多种范式和 API 才能编写应用程序。



Pulsar is equipped for the queue and event stream models because of its modular design. In [Chapter 3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#pulsar), we will walk through all the components of Pulsar in depth, but to motivate this discussion it’s worth talking about some of them now. Pulsar’s design keeps a clear separation between the various responsibilities of the system.

由于 Pulsar 采用了模块化设计，所以同时具备了队列和事件流模型。在 [第 3 章](./ch03-pulsar.md)中，我们将深入了解 Pulsar 的所有组件，现在我们先来窥一斑。 Pulsar 的设计在系统的各种职责之间保持了清晰的界限。



The following are some of Pulsar’s responsibilities:

- Storing data for finite periods for consumers
- Storing data for long periods for consumers
- Ensuring order in the topics

以下是 Pulsar 的一些职责：

- 为消费者存储有限时期的数据
- 为消费者长期存储数据
- 确保主题的顺序



Pulsar’s short-term storage is managed by the Pulsar brokers. Pulsar’s long-term storage is handled by Apache BookKeeper. These choices enable a rich experience and make Pulsar suitable for a wide range of problems in the messaging space.

Pulsar 的短期存储由 Pulsar Broker 管理。 Pulsar 的长期存储由 Apache BookKeeper 处理。这些选择带来了丰富的体验，并使 Pulsar 适用于消息投递领域中的各种问题。



For a mature company, migrating from an existing messaging system like RabbitMQ, MQTT, or Kafka to Pulsar may be infeasible. Each of these platforms has a unique protocol, requires custom client libraries, and has a unique paradigm and vernacular. The process of migrating may take years for a sufficiently large organization. Fortunately, Pulsar can be used concurrently with these existing messaging systems, allowing organizations to use, say, Pulsar and RabbitMQ at the same time and slowly migrate their RabbitMQ topics to Pulsar, or keep both running side by side through the Pulsar bridge framework.[^iii] The Pulsar bridge framework provides a mechanism to translate messages from AMQP 0.9.1 (the protocol RabbitMQ uses) to Pulsar. In this model, the RabbitMQ applications can continue to use RabbitMQ and their AMQP 0.9.1 messages will convert to Pulsar protocol messages in the background. When the team is ready, they can start to consume their RabbitMQ messages from Pulsar where they left off.

对于一家成熟的公司，从现有的消息系统（如 RabbitMQ、MQTT 或 Kafka）迁移到 Pulsar 可能是不可行的。每个平台都有自己独特的协议，需要自定义客户端库，并且有一个独特的范式和方言。对于足够大的组织而言，迁移过程可能需要数年时间。幸运的是，Pulsar 可以与这些现有的消息传递系统同时使用，允许组织同时使用 Pulsar 和 RabbitMQ，并将他们的 RabbitMQ 主题慢慢迁移到 Pulsar，或者通过 Pulsar 桥接框架让两者并行运行。 [^iii] Pulsar 桥接框架提供了一种将消息从 AMQP 0.9.1（RabbitMQ 使用的协议）转换为 Pulsar 的机制。在此模型中，RabbitMQ 应用程序可以继续使用 RabbitMQ，其 AMQP 0.9.1 消息将在后台转换为 Pulsar 协议消息。当团队准备好后，他们可以开始从 Pulsar 中消费 RabbitMQ 消息。

[^iii]: We will cover Pulsar bridges in [Chapter 12](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch12.html#operating_pulsar).



The power of Pulsar’s modular design is also evident in its ecosystem. Pulsar supports Functions as a Service (see [Figure 2-5](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#pulsar_functions_are_a_built-in_runtime)), as well as the ability to use SQL with Pulsar topic data and change data capture (CDC) with minimal configuration. Each of these features provides additional building blocks and tools for creating rich, event-driven applications.

Pulsar 模块化设计的力量也体现在其生态系统中。 Pulsar 支持 Functions as a Service（参见 图 2-5），还能够以最少的配置将 SQL 与 Pulsar 主题数据和变更数据捕获 (CDC) 一起使用。 这些特性为创建丰富的事件驱动应用程序提供了额外的构建块和工具。

![Pulsar Functions are a built-in runtime for stream processing in Pulsar.](../img/mapu_0205.png)

*图 2-5. Pulsar Functions 是 Pulsar 中内置的流式处理运行时。*



## 性能

Thus far, we have discussed three critical components of a quality event broker. The broker needs to 1) reliably store data, 2) reliably deliver messages to consumers, and 3) quickly consume messages from publishers. Performing these three tasks well requires thoughtful design and optimized resource utilization. All event brokers have to work through the same disk speed, CPU, memory, and network limitations. In further chapters, we’ll get into more detail around the design considerations in Apache Pulsar, but for now let’s take a look at how some of them enable exceptional performance and scalability.

到目前为止，我们已经讨论了优秀的事件 Broker 有三个关键任务：1) 可靠地存储数据，2) 可靠地将消息传递给消费者，3) 快速消费来自发布者的消息。做好这三项任务需要深思熟虑地设计以及优化资源利用。所有事件 Broker都必须在相同的磁盘速度、CPU、内存和网络限制下工作。在接下来的章节中，我们将详细介绍 Apache Pulsar 中的设计考量因素，但现在我们看看其中一些设计考量是如何实现卓越性能和可扩展性的。



In [“Modularity”](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch02.html#modularity-id000009), we discussed Pulsar’s modular approach to storage by using Apache BookKeeper. We focused on how this choice enables features to archive and retrieve Pulsar data. Pulsar administrators can grow the size of the BookKeeper cluster separately from the Pulsar event broker nodes. The storage needs may change during a day, month, or year within a messaging platform. Pulsar enables the flexibility to scale up storage more comfortably with this design decision. When it comes to reliability concerns about storing data, the storage systems’ scalability is a significant factor.

在模块化一节中，我们讨论了 Pulsar 使用 Apache BookKeeper 进行存储的模块化方法。我们重点讨论了这种选择是如何实现归档和检索 Pulsar 数据功能的。 Pulsar 管理员可以独立于 Pulsar Broker 节点增加 BookKeeper 集群的大小。在消息平台内，存储需求可能会在一天、一个月或一年内发生变化。通过这一设计决策，Pulsar 能够更灵活地扩展存储。当谈到存储数据的可靠性问题时，存储系统的可扩展性是一个重要因素。



Reliability in the consumption of messages is contingent on the event broker being able to consume the volume of messages sent its way. If an event broker can’t keep up with the volume of messages, many failure scenarios may follow. Clients connect to Pulsar via the Pulsar protocol and connect to a Pulsar node. Since Pulsar nodes can scale separately from the BookKeeper cluster, scaling up consumption is also more flexible.

消息消费的可靠性取决于事件 Broker 能够消费其发送的消息量。如果事件 Broker 无法跟上消息量，则可能会出现许多故障情况。客户端通过 Pulsar 协议连接到 Pulsar 并连接到 Pulsar 节点。由于 Pulsar 节点可以与 BookKeeper 集群分开扩展，因此扩展消费能力也更加灵活。



Finally, what about raw performance? How many messages can a Pulsar cluster consume per second? How many can it securely store in the BookKeeper cluster per second? There are many published benchmarks[^iiii] on Apache Pulsar and its performance, but you should take every benchmark with a grain of salt. As mentioned earlier in this chapter, every messaging system has constraints. The engineers who design these systems take advantage of their unique knowledge and circumstances. Therefore, designing benchmarks that fairly assess the performance of each platform is often an exercise in futility. That said, Apache Pulsar has a reputation for being a performant platform, and hundreds of companies have chosen Pulsar to manage their event streaming platforms.

最后，原始性能如何？ Pulsar 集群每秒可以消耗多少条消息？它每秒可以安全地存储多少个 BookKeeper 集群？关于 Apache Pulsar 及其性能有许多已发布的基准测试[^iiii]，但您应该对每个基准测试持保留态度。正如本章前面提到的，每个消系统都有限制。设计这些系统的工程师利用了他们独特的知识和环境。因此，设计公平评估每个平台性能的基准通常是徒劳的。也就是说，Apache Pulsar 以高性能平台而闻名，数百家公司选择 Pulsar 来管理他们的事件流平台。

[^iiii]: For example, see [*Benchmarking Apache Kafka, Apache Pulsar, and RabbitMQ: Which Is the Fastest?*](https://oreil.ly/b67QJ); [*Benchmarking Pulsar and Kafka—A More Accurate Perspective on Pulsar’s Performance*](https://oreil.ly/uewER); and [*Performance Comparison Between Apache Pulsar and Kafka: Latency*](https://oreil.ly/u4DpP).

# 总结

In this chapter you acquired the foundational knowledge needed to understand Pulsar’s value proposition and uniqueness. From here, we’ll pull apart all of Pulsar’s components to gain a deep understanding of the basic building blocks. With that knowledge, you’ll be ready to dive deep into the APIs and start building applications.

在本章中，我们讲解了理解 Pulsar 的价值主张和独特性所需的基础知识。 从本章开始，我们将分解 Pulsar 的所有组件，以深入了解基本构建块。 有了这些知识，您就可以深入了解 API 并开始构建应用程序。
