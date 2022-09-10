# Preface

# Why I Wrote This Book

Throughout my career, I’ve been tasked with learning complex systems as part of my job. Early on I had to learn how to write MapReduce jobs and understand the intricacies of the Hadoop Distributed File System (HDFS) and the Hadoop ecosystem; years later I learned early versions of Apache Spark. Today I’m still tasked with learning about complex systems for my job. Over the years, well-written technical blog posts, articles, and books have been instrumental in my ability to learn and to apply what I learn at work. With this book, I sought to create a resource that would provide a thorough explanation of the value of Apache Pulsar which could be long-lasting and fun.

Along with Apache Pulsar as a technology with its trade-offs and consideration is a broader ecosystem and ideas of event streaming. This book provides a nurturing environment to work through the event streams paradigm and provide the reader with context and a road map for adopting event streaming architectures.

# Who This Book Is For

This book is targeted at two audiences: those who want to learn about Apache Pulsar and those who are curious about event streaming architectures. For the first audience, this book provides a thorough overview of Apache Pulsar, all of its components, and code samples for getting started with Pulsar and its ecosystem. For the second audience, it serves as a primer for adding Apache Pulsar, Apache Kafka, or another event streaming technology to your architecture.

# How I Organized This Book

I spend Chapters [1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch01.html#the_value_of_real-time_messaging) through [3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch03.html#pulsar) explaining the motivation for Apache Pulsar and the rise of event streams, as well as provide the reader with more supporting content. In Chapters [4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pulsar_internals) through [10](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch10.html#pulsar_sql-id000029), I dive deep into the internals of Pulsar, component by component, to give the reader a complete understanding of how Pulsar works. I finish the book by focusing on the operational considerations of Pulsar. Chapters [11](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch11.html#deploying_pulsar) and [12](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch12.html#operating_pulsar) take a detailed look at deploying Pulsar and operating Pulsar in production. These chapters dive deeper into what Pulsar looks like when deployed on systems like Kubernetes and what metrics are available for use as an operator. In [Chapter 13](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch13.html#the_future), I imagine what the future of Pulsar will look like in 3–5 years, including ways the project can expand to meet the growing needs of the community. Finally, Appendices [A](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app01.html#pulsar_admin_api) through [D](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app04.html#d_securitycomma_authenticationcomma_and) cover topics like Admin APIs, Security, and GeoReplication. I believe that organizing the book in this way will give the reader the best experience reading the book end to end as well as using it as a reference manual if needed.

# Conventions Used in This Book

The following typographical conventions are used in this book:

- *Italic*

  Indicates new terms, URLs, email addresses, filenames, and file extensions.

- `Constant width`

  Used for program listings, as well as within paragraphs to refer to program elements such as variable or function names, databases, data types, environment variables, statements, and keywords.

- **`Constant width bold`**

  Shows commands or other text that should be typed literally by the user.

- *`Constant width italic`*

  Shows text that should be replaced with user-supplied values or by values determined by context.

# Using Code Examples

Supplemental material (code examples, exercises, etc.) is available for download at [*http://www.github.com/josep2*](http://www.github.com/josep2).

If you have a technical question or a problem using the code examples, please send email to [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com).

This book is here to help you get your job done. In general, if example code is offered with this book, you may use it in your programs and documentation. You do not need to contact us for permission unless you’re reproducing a significant portion of the code. For example, writing a program that uses several chunks of code from this book does not require permission. Selling or distributing examples from O’Reilly books does require permission. Answering a question by citing this book and quoting example code does not require permission. Incorporating a significant amount of example code from this book into your product’s documentation does require permission.

We appreciate, but generally do not require, attribution. An attribution usually includes the title, author, publisher, and ISBN. For example: “*Mastering Apache Pulsar* by Jowanza Joseph (O’Reilly). Copyright © 2022 Jowanza Joseph, 978-1-492-08490-7.”

If you feel your use of code examples falls outside fair use or the permission given above, feel free to contact us at [*permissions@oreilly.com*](mailto:permissions@oreilly.com).

# O’Reilly Online Learning

> **NOTE**
>
> For more than 40 years, [*O’Reilly Media*](http://oreilly.com/) has provided technology and business training, knowledge, and insight to help companies succeed.

Our unique network of experts and innovators share their knowledge and expertise through books, articles, and our online learning platform. O’Reilly’s online learning platform gives you on-demand access to live training courses, in-depth learning paths, interactive coding environments, and a vast collection of text and video from O’Reilly and 200+ other publishers. For more information, visit [*http://oreilly.com*](http://oreilly.com/).

# How to Contact Us

Please address comments and questions concerning this book to the publisher:

- O’Reilly Media, Inc.
- 1005 Gravenstein Highway North
- Sebastopol, CA 95472
- 800-998-9938 (in the United States or Canada)
- 707-829-0515 (international or local)
- 707-829-0104 (fax)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [*https://oreil.ly/mastering-apache-pulsar*](https://oreil.ly/mastering-apache-pulsar).

Email [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com) to comment or ask technical questions about this book.

For news and information about our books and courses, visit [*http://oreilly.com*](http://oreilly.com/).

Find us on Facebook: [*http://facebook.com/oreilly*](http://facebook.com/oreilly)

Follow us on Twitter: [*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

Watch us on YouTube: [*http://youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# Acknowledgments

First, I want to thank the creators and maintainers of the open source Apache Pulsar project. Their work brought the project to fruition, and without it, this book would not exist. I also want to thank the editorial and content acquisition teams at O’Reilly. They challenged me to make this book as good as possible, and it would only be a fraction as good without their work. I would also like to thank my wife, Bethany. She provided all the illustrations for this book and supported me through the year I spent writing it. Finally, I’d like to thank the technical editors who provided the invaluable feedback that made this book possible.