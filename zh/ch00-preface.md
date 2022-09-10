# 前言

# 我为什么写这本书

Throughout my career, I’ve been tasked with learning complex systems as part of my job. Early on I had to learn how to write MapReduce jobs and understand the intricacies of the Hadoop Distributed File System (HDFS) and the Hadoop ecosystem; years later I learned early versions of Apache Spark. Today I’m still tasked with learning about complex systems for my job. Over the years, well-written technical blog posts, articles, and books have been instrumental in my ability to learn and to apply what I learn at work. With this book, I sought to create a resource that would provide a thorough explanation of the value of Apache Pulsar which could be long-lasting and fun.

在我的职业生涯中，学习复杂系统的任务一直是我工作的一部分。早期我必须学习如何编写 MapReduce 作业，了解 Hadoop 分布式文件系统（HDFS）和 Hadoop 生态系统的复杂性；多年后，我学会了 Apache Spark 的早期版本。今天，我的任务仍然是为我的工作学习复杂系统。多年来，写得很好的技术博客文章、文章和书籍对我学习和在工作中应用所学知识的能力起到了很大作用。通过这本书，我试图创造一种资源，对 Apache Pulsar 的价值进行透彻的解释，这种解释可以是持久的、有趣的。



Along with Apache Pulsar as a technology with its trade-offs and consideration is a broader ecosystem and ideas of event streaming. This book provides a nurturing environment to work through the event streams paradigm and provide the reader with context and a road map for adopting event streaming architectures.

伴随着 Apache Pulsar 作为一项技术的取舍和考虑，是一个更广泛的生态系统和事件流的想法。本书提供了一个培养环境，通过事件流范式的工作，为读者提供背景和采用事件流架构的路线图。



# 本书为谁而写

This book is targeted at two audiences: those who want to learn about Apache Pulsar and those who are curious about event streaming architectures. For the first audience, this book provides a thorough overview of Apache Pulsar, all of its components, and code samples for getting started with Pulsar and its ecosystem. For the second audience, it serves as a primer for adding Apache Pulsar, Apache Kafka, or another event streaming technology to your architecture.

本书针对两种读者：想了解 Apache Pulsar 的人和对事件流架构感到好奇的人。对于第一类读者，本书提供了对 Apache Pulsar、其所有组件和代码样本的全面概述，以便开始使用Pulsar和其生态系统。对于第二类读者，本书是将 Apache Pulsar、Apache Kafka或 其他事件流技术添加到你的架构中的入门读物。



# 我是如何组织这本书的

I spend Chapters [1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch01.html#the_value_of_real-time_messaging) through [3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#pulsar) explaining the motivation for Apache Pulsar and the rise of event streams, as well as provide the reader with more supporting content. In Chapters [4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pulsar_internals) through [10](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch10.html#pulsar_sql-id000029), I dive deep into the internals of Pulsar, component by component, to give the reader a complete understanding of how Pulsar works. I finish the book by focusing on the operational considerations of Pulsar. Chapters [11](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch11.html#deploying_pulsar) and [12](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch12.html#operating_pulsar) take a detailed look at deploying Pulsar and operating Pulsar in production. These chapters dive deeper into what Pulsar looks like when deployed on systems like Kubernetes and what metrics are available for use as an operator. In [Chapter 13](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch13.html#the_future), I imagine what the future of Pulsar will look like in 3–5 years, including ways the project can expand to meet the growing needs of the community. Finally, Appendices [A](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app01.html#pulsar_admin_api) through [D](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app04.html#d_securitycomma_authenticationcomma_and) cover topics like Admin APIs, Security, and GeoReplication. I believe that organizing the book in this way will give the reader the best experience reading the book end to end as well as using it as a reference manual if needed.

我在第[1](./ch01-the_value_of_real-time_messaging.md)至[3](./ch03-pulsar.md)章中解释了 Apache Pulsar 的动机和事件流的兴起，并为读者提供了更多的背景知识。在第[4](./ch04-pulsar_internals.md)至[10](./ch10-pulsar_sql.md)章中，我逐一深入到 Pulsar 的内部，让读者全面了解 Pulsar 的工作原理。在本书的最后，我重点介绍了 Pulsar 的操作注意事项。第[11](./ch11-deploying_pulsar.md)章和第[12](./ch12-operating_pulsar.md)章详细介绍了在生产中部署 Pulsar 和操作 Pulsar 。这些章节深入探讨了 Pulsar 在 Kubernetes 等系统上部署时的情况，以及作为运维人员可以使用哪些指标。在第[13](./ch13-the_future.md)章中，我想象了3-5年后 Pulsar 的未来是什么样子，包括该项目可以如何扩展以满足社区日益增长的需求。最后，附录[A](./appendix-a_pulsar_admin_api.md)至[D](./appendix-d_auth.md)涵盖了管理API、安全和地理复制等主题。我相信，以这种方式组织本书，将给读者带来从头到尾阅读本书的最佳体验，并在需要时将其作为参考手册。



# 本书中使用的惯例

本书中使用了以下排版惯例。

- *斜体*

  表示新术语、URL、电子邮件地址、文件名和文件扩展名。

- `等宽字体`
  用于程序列表，以及在段落中指代程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键词。
  
- **`加粗等宽字体`**

  显示命令或其他应该由用户直接输入的文本。

- *`斜体等宽字体`*
显示应该用用户提供的值或由上下文决定的值来替换的文本。



# 使用代码示例

补充材料（代码实例、练习等）可在 [*http://www.github.com/josep2*](http://www.github.com/josep2) 下载。

如果你有技术问题或在使用代码实例时遇到问题，请发电子邮件到[*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com)。

本书可帮助你完成你的工作。一般来说，你可以在程序和文档中使用本书提供的示例代码。你不需要联系我们获得许可，除非你要复制代码的很大一部分。例如，编写一个使用本书几大块代码的程序不需要许可。销售或分发O'Reilly 书中的例子则需要许可。通过引用本书和引用示例代码来回答一个问题不需要许可。将本书中的大量示例代码纳入你的产品文档中，需要得到许可。

我们感谢，但一般不要求署名。署名通常包括标题、作者、出版商和ISBN。比如说。"*掌握 Apache Pulsar*，Jowanza Joseph（O'Reilly）撰写。Copyright © 2022 Jowanza Joseph, 978-1-492-08490-7."

如果你觉得你对代码实例的使用超出了合理使用或上述许可的范围，请随时与我们联系：[*permissions@oreilly.com*](mailto:permissions@oreilly.com)。



# O’Reilly 在线学习

> **注意**
>
> 40多年来，[*O'Reilly Media*](http://oreilly.com/) 提供技术和商业培训、知识和洞察力，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O'Reilly 在线学习平台让你可以按需访问现场培训课程、深入学习路径、互动编码环境以及来自 O'Reilly 和其他 200 多家出版商的大量文本和视频。欲了解更多信息，请访问 [*http://oreilly.com*](http://oreilly.com/)。



# 如何联系我们

有关本书的意见和问题，请向出版商提出。

- O'Reilly Media, Inc.
- 1005 Gravenstein Highway North
- Sebastopol, CA 95472
- 800-998-9938 (美国或加拿大境内)
- 707-829-0515 (国际或当地)
- 707-829-0104 (传真)

我们为这本书建立了一个网页，在那里我们列出了勘误表、例子和任何其他信息。你可以通过 [*https://oreil.ly/mastering-apache-pulsar*](https://oreil.ly/mastering-apache-pulsar) 访问这个页面。

发送电子邮件至 [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com) 来评论或询问有关本书的技术问题。

有关我们书籍和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com/)。

在Facebook上找到我们。[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在Twitter上关注我们。[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在YouTube上观看我们。[*http://youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)



# 鸣谢

首先，我想感谢 Apache Pulsar 开源项目的创建者和维护者。他们的工作使该项目取得了成果，没有它，这本书就不会存在。我也要感谢 O'Reilly 公司的编辑和内容获取团队。他们向我提出挑战，让我把这本书写得尽可能好，如果没有他们的工作，这本书只会有一小部分好。我还要感谢我的妻子，贝瑟妮。她为本书提供了所有的插图，并在我写这本书的一年中一直支持着我。最后，我要感谢技术编辑，他们提供了宝贵的反馈意见，使本书得以出版。