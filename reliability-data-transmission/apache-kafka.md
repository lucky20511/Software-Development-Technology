From: [https://techbeacon.com/what-apache-kafka-why-it-so-popular-should-you-use-it](https://techbeacon.com/what-apache-kafka-why-it-so-popular-should-you-use-it)

## What is Apache Kafka? Why is it so popular? Should you use it?

RedMonk.com published [an article](https://redmonk.com/fryan/2016/02/04/the-rise-and-rise-of-apache-kafka/) in February 2016 documenting some interesting stats around the "rise and rise" of a powerful asynchronous messaging technology called[Apache Kafka](http://kafka.apache.org/).

If you're unfamiliar with Kafka, it's a scalable, fault-tolerant, publish-subscribe messaging system that enables you to build distributed applications and powers web-scale Internet companies such as LinkedIn, Twitter, AirBnB, and many others. Small-scale open-source projects come and go, but it seems Kafka has some really strong momentum behind it.

So what is the value of Kafka? Why is it suddenly so popular, and should _you_ use it? I'll give you enough information in this article to answer these questions.

## Even slow-to-evolve enterprises are noticing Kafka

RedMonk points out that Apache Kafka-related questions on StackOverflow, Apache Kafka trends on Google, and Kafka GitHub stars are all shooting up. In the graph below, you can see that GitHub interest has grown exponentially:

### Apache Kafka GitHub Stars Growth

![](http://techbeacon.com/sites/default/files/screen_shot_2016-03-09_at_7_16_50_pm.png)

_Image credit:_[_RedMonk_](http://redmonk.com/fryan/files/2016/02/apache-kafka-github-stars.jpg)

This kind of technology is not only for Internet unicorns. I meet with enterprise architects every week, and I've noticed that Kafka has made a noticeable impact on typically slower-to-adopt, traditional enterprises as well. My colleagues in the enterprise and I are starting to see a common trend across companies of all backgrounds. They are starting to realize that to build the digital services that will disrupt and innovate, they need access to a wide stream of data, and that data must be integrated. However, the typical source of data—transactional data such as orders, inventory, and shopping carts — is being augmented with things such as page clicks, "likes," recommendations, and searches. All of this data is deeply important to understanding customers' behaviors and frictions, and it can feed a set of predictive analytics engines that can be the differentiator for companies. This is where Kafka comes in.

## Kafka's origin story at LinkedIn

Kafka was developed around 2010 at LinkedIn by a team that included Jay Kreps, Jun Rao, and Neha Narkhede. The problem they originally set out to solve was low-latency ingestion of large amounts of event data from the LinkedIn website and infrastructure into a[lambda architecture](http://lambda-architecture.net/)that harnessed Hadoop and real-time event processing systems. The key was the "real-time" processing. At the time, there weren't any solutions for this type of ingress for real-time applications.

There were good solutions for ingesting data into offline batch systems, but they exposed implementation details to downstream users and used a push model that could easily overwhelm a consumer. Also, they were not designed for the real-time use case.

Real-time systems such as the traditional messaging queues \(think[ActiveMQ](http://activemq.apache.org/),[RabbitMQ](https://www.rabbitmq.com/), etc.\) have great delivery guarantees and support things such as transactions, protocol mediation, and message consumption tracking, but they are overkill for the use case LinkedIn had in mind. Everyone \(including LinkedIn\) wants to build fancy machine-learning algorithms, but without the data, the algorithms are useless. Getting the data from source systems and reliably moving it around was very difficult, and existing batch-based solutions and enterprise messaging solutions did not solve the problem.

Kafka was developed to be the ingestion backbone for this type of use case. Back in 2011, Kafka was ingesting more than 1 billion events a day. Recently, LinkedIn has[reported ingestion rates of 1 trillion messages a day](http://www.confluent.io/blog/apache-kafka-hits-1.1-trillion-messages-per-day-joins-the-4-comma-club). Let's take a deeper look at what Kafka is and how it is able to handle these use cases.

## How does Kafka work?

Kafka looks and feels like a publish-subscribe system that can deliver in-order, persistent, scalable messaging. It has publishers, topics, and subscribers. It can also partition topics and enable massively parallel consumption. All messages written to Kafka are persisted and replicated to peer brokers for fault tolerance, and those messages stay around for a configurable period of time \(i.e., 7 days, 30 days, etc.\).

The key to Kafka is the_log_. Developers often get confused when first hearing about this "log," because we're used to understanding "logs" in terms of application logs. What we're talking about here, however, is the_log data structure_. The log is simply a time-ordered, append-only sequence of data inserts where the data can be anything \(in Kafka, it's just an array of bytes\). If this sounds like the basic data structure upon which a database is built, it is.

![](http://techbeacon.com/sites/default/files/kafka_log.png)

_Image credit:_[_Apache Kafka_](https://camo.githubusercontent.com/211c39f3c48111efebaa1d3422e7c94539782d1f/68747470733a2f2f6b61666b612e6170616368652e6f72672f696d616765732f6b61666b615f6c6f672e706e67)

Databases write change events to a log and derive the value of columns from that log. In Kafka, messages are written to a\_topic,\_which maintains this log \(or multiple logs — one for each partition\) from which subscribers can read and derive their own representations of the data \(think[materialized view](https://en.wikipedia.org/wiki/Materialized_view)\).

For example, a "log" of the activity for a shopping cart could include "add item foo," "add item bar," "remove item foo," and "checkout." The log would present these facts to downstream systems. If a shopping cart service reads that log, it can derive the shopping cart objects that represent what's in the shopping cart: item "bar" and ready for checkout. Because Kafka can retain messages for a long time \(or forever\), applications can rewind to old positions in the log and reprocess. Think of the situation where you want to come up with a new application or new analytic algorithm \(or change an existing one\) and test it out against past events.

## What Kafka doesn't do

Kafka can be very fast because it presents the log data structure as a first-class citizen. It's not a traditional message broker with lots of bells and whistles.

* Kafka does not have individual message IDs. Messages are simply addressed by their offset in the log.
* Kafka also does not track the consumers that a topic has or who has consumed what messages. All of that is left up to the consumers.

Because of those differences from traditional messaging brokers, Kafka can make optimizations.

* It lightens the load by not maintaining any indexes that record what messages it has. There is no random access — consumers just specify offsets and Kafka delivers the messages in order, starting with the offset.
* There are no deletes. Kafka keeps all parts of the log for the specified time.
* It can efficiently stream the messages to consumers using kernel-level IO and not buffering the messages in user space.
* It can leverage the operating system for file page caches and efficient writeback/writethrough to disk.

## Kafka and big data at web-scale companies

Because of these performance characteristics and its scalability, Kafka is used heavily in the big data space as a reliable way to ingest and move large amounts of data very quickly. For example,[Netflix started out writing its own ingestion framework](http://techblog.netflix.com/2016/02/evolution-of-netflix-data-pipeline.html)that dumped data into Amazon S3 and used Hadoop to run batch analytics of video streams, UI activities, performance events, and diagnostic events to help drive feedback about user experience. As the demand for real-time \(sub-minute\) analytics grew, Netflix moved to using Kafka as its primary backbone for ingestion via Java APIs or REST APIs. Netflix's system now supports ingestion of ~500 billion events per day \(~1.3 PB data\) and at peak up to ~8 million events per second. It has paired Kafka with streaming stacks like[Apache Spark](http://spark.apache.org/)and[Apache Samza](http://samza.apache.org/) to route data and load it into back-end data stores like ElasticSearch and Cassandra, as well as directly into real-time analytics engines.

### Big Data ingestion at Netflix![](http://techbeacon.com/sites/default/files/spark-and-spark-streaming-at-netfixkedar-sedekar-and-monal-daxini-netflix-21-638.jpg)

_Image credit:_[_Netflix_](http://www.slideshare.net/SparkSummit/spark-and-spark-streaming-at-netfix-sedakar-daxini)

This architecture is new alternative to the lambda architecture, and some are calling it the  [kappa architecture](http://kappa-architecture.com/). Open-source developers are integrating Kafka with other interesting tools. One stack, called [SMACK](http://www.slideshare.net/akirillov/data-processing-platforms-architectures-with-spark-mesos-akka-cassandra-and-kafka), combines Apache Spark, Apache Mesos, Akka, Cassandra, and Kafka to implement a type of[CQRS \(command query responsibility separation\)](http://martinfowler.com/bliki/CQRS.html). This stack benefits from powerful ingestion \(Kafka\), back-end storage for write-intensive apps \(Cassandra\), and replication to a more query-intensive set of apps \(Cassandra again\). All of this can be managed with a resource/cluster management solution such as Apache Mesos.

## How Kafka supports microservices

As powerful and popular as Kafka is for big data ingestion, the "log" data structure has interesting implications for applications built around the Internet of Things, microservices, and cloud-native architectures in general.[Domain-driven design](http://techbeacon.com/why-you-need-domain-driven-design)concepts like CQRS and[event sourcing](http://martinfowler.com/eaaDev/EventSourcing.html)are powerful mechanisms for implementing[scalable microservices](http://techbeacon.com/challenges-scaling-microservices), and Kafka can provide the backing store for these concepts. Event sourcing applications that generate a lot of events can be difficult to implement with traditional databases, and an additional feature in Kafka called "log compaction" can preserve events for the lifetime of the app. Basically, with log compaction, instead of discarding the log at preconfigured time intervals \(7 days, 30 days, etc.\), Kafka can keep the entire set of recent events around for all the keys in the set. This helps make the application very loosely coupled, because it can lose or discard logs and just restore the domain state from a log of preserved events.

## How does Kafka compare to traditional messaging competitors?

Just as the evolution of the database from RDBMS to specialized stores has led to efficient technology for the problems that need it, messaging systems have evolved from the "one size fits all" message queues to more nuanced implementations \(or assumptions\) for certain classes of problems. Both Kafka and traditional messaging have their place.

Traditional message brokers allow you to keep consumers fairly simple in terms of reliable messaging guarantees. The broker \(JMS, AMQP, or whatever\) tracks what messages have been acknowledged by the consumer and can help a lot when order processing guarantees are required and messages must not be missed. Traditional brokers typically implement multiple protocols \(e.g., Apache ActiveMQ implements AMQP, MQTT, STOMP, and others\) to be used as a bridge for components that use different protocols. Additional functionalities such as message TTLs, non-persistent messaging, request-response messaging, correlation ID selectors, etc. are all perfectly valid messaging use cases where Kafka would not be a good fit.

## Should you use Kafka?

The answer will always depend on what your use case is. Kafka fits a class of problem that a lot of web-scale companies and enterprises have, but just as the traditional message broker is not a one size fits all, neither is Kafka. If you're looking to build a set of resilient data services and applications, Kafka can serve as the source of truth by collecting and keeping all of the "facts" or "events" for a system.

In the end, you'll have to consider the trade-offs and drawbacks. If you think you can benefit from having multiple publish/subscribe and queueing tools, it might be worth considering.

