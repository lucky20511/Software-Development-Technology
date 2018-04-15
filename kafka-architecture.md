# Kafka Architecture

First, if you are not sure what Kafka is, see [this article](http://cloudurable.com/blog/what-is-kafka/index.html).

Kafka consists of records, topics, consumers, producers, brokers, logs, partitions, and clusters. Records can have keys \(optional\), values, and timestamps. Kafka records are immutable. A Kafka Topic is a stream of records \(`"/orders"`, `"/user-signups"`\). You can think of a topic as a feed name. A topic has a log which is the topic’s storage on disk. A topic log is broken up into partitions and segments. The Kafka Producer API is used to produce streams of data records. The Kafka Consumer API is used to consume a stream of records from Kafka. A broker is a Kafka server that runs in a Kafka cluster. Kafka brokers form a cluster. The Kafka cluster consists of many Kafka brokers on many servers. Broker sometimes refer to more of a logical system or as Kafka as a whole.

[Jean-Paul Azar](https://www.linkedin.com/in/jean-paul-azar-430ba5132/) works at [Cloudurable](http://cloudurable.com/). Cloudurable provides [Kafka training](http://cloudurable.com/kafka-training/index.html), [Kafka consulting](http://cloudurable.com/kafka-aws-consulting/index.html), [Kafka support](http://cloudurable.com/subscription_support/index.html) and helps [setting up Kafka clusters in AWS](http://cloudurable.com/services/index.html).

## Kafka Architecture: Topics, Producers, and Consumers {#kafka-architecture-topics-producers-and-consumers}

![](http://cloudurable.com/images/kafka-architecture-topics-producers-consumers.png "Kafka Architecture - Topics, Producers and Consumers Diagram")

Kafka uses ZooKeeper to manage the cluster. ZooKeeper is used to coordinate the brokers/cluster topology. ZooKeeper is a consistent file system for configuration information. ZooKeeper is used for leadership election for broker topic partition leaders.

## Kafka Architecture: Core Kafka {#kafka-architecture-core-kafka}

![](http://cloudurable.com/images/kafka-architecture-core-kafka.svg "Kafka Architecture - Core Kafka Diagram")

## Kafka Needs ZooKeeper {#kafka-needs-zookeeper}

Kafka uses ZooKeeper to do leadership election of Kafka broker and topic partition pairs. Kafka uses ZooKeeper to manage service discovery for Kafka brokers that form the cluster. ZooKeeper sends changes of the topology to Kafka, so each node in the cluster knows when a new broker joins, a Broker dies, a topic was removed or a topic was added, etc. ZooKeeper provides an in-sync view of Kafka Cluster configuration.

## Kafka Producer, Consumer, Topic Details {#kafka-producer-consumer-topic-details}

Kafka producers write to Topics. Kafka consumers read from Topics. A topic is associated with a log which is data structure on disk. Kafka appends records from a producer\(s\) to the end of a topic log. A topic log consists of many partitions that are spread over multiple files which can be spread on multiple Kafka cluster nodes. Consumers read from Kafka topics at their cadence and can pick where they are \(offset\) in the topic log. Each consumer group tracks offset from where they left off reading. Kafka distributes topic log partitions on different nodes in a cluster for high performance with horizontal scalability. Spreading partitions aids in writing data quickly. Topic log partitions are Kafka way to shard reads and writes to the topic log. Also, partitions are needed to have multiple consumers in a consumer group work at the same time. Kafka replicates partitions to many nodes to provide failover.

## Kafka Architecture: Topic Partition, Consumer Group, Offset, and Producers {#kafka-architecture-topic-partition-consumer-group-offset-and-producers}

![](http://cloudurable.com/images/kafka-architecture-topic-partition-consumer-group-offset-producers.png "Kafka Architecture: Topic Partition, Consumer group, Offset and Producers Diagram")

Take a look at the following illustration. It shows the cluster diagram of Kafka.

![](https://www.tutorialspoint.com/apache_kafka/images/cluster_architecture.jpg "Cluster Architecture")

### Kafka Scale and Speed {#kafka-scale-and-speed}

How can Kafka scale if multiple _producers_ and _consumers_ read and write to same Kafka topic log at the same time? First Kafka is fast, Kafka writes to filesystem sequentially, which is fast. On a modern fast drive, Kafka can easily write up to 700 MB or more bytes of data a second. Kafka scales writes and reads by _sharding topic logs into partitions_. Recall topics logs can be split into multiple partitions which can be stored on multiple different servers, and those servers can use multiple disks. Multiple producers can write to different _partitions \_of the same topic. Multiple consumers from multiple \_consumer groups_ can read from different partitions efficiently.

### Kafka Brokers {#kafka-brokers}

A _Kafka cluster_ is made up of multiple Kafka Brokers. Each Kafka Broker has a unique ID \(number\). Kafka Brokers contain topic log partitions. Connecting to one broker bootstraps a client to the entire Kafka cluster. For failover, you want to start with at least three to five brokers. A Kafka cluster can have, 10, 100, or 1,000 brokers in a cluster if needed.

### Kafka Cluster, Failover, ISRs {#kafka-cluster-failover-isrs}

Kafka supports replication to support failover. Recall that Kafka uses ZooKeeper to form Kafka Brokers into a cluster and each node in Kafka cluster is called a Kafka Broker. Topic partitions can be replicated across multiple nodes for failover. The topic should have a replication factor greater than 1 \(2, or 3\). For example, if you are running in AWS, you would want to be able to survive a single availability zone outage. If one Kafka Broker goes down, then the Kafka Broker which is an ISR \(in-sync replica\) can serve data.

### Kafka Failover vs. Kafka Disaster Recovery {#kafka-failover-vs-kafka-disaster-recovery}

Kafka uses replication for failover. Replication of Kafka topic log partitions allows for failure of a rack or AWS availability zone \(AZ\). You need a replication factor of at least 3 to survive a single AZ failure. You need to use Mirror Maker, a Kafka utility that ships with Kafka core, for disaster recovery. Mirror Maker replicates a Kafka cluster to another datacenter or AWS region. They call what Mirror Maker does mirroring as not to be confused with replication.

Note that there is no hard and fast rule on how you have to set up the Kafka cluster per se. You could, for example, set up the whole cluster in a single AZ so you can use AWS enhanced networking and placement groups for higher throughput, and then use Mirror Maker to mirror the cluster to another AZ in the same region as a hot-standby.

## Kafka Architecture: Kafka Zookeeper Coordination {#kafka-architecture-kafka-zookeeper-coordination}

![](http://cloudurable.com/images/kafka-architecture-kafka-zookeeper-coordination.png "Kafka Architecture - Kafka Zookeeper Coordination Diagram")

### Kafka Topics Architecture {#kafka-topics-architecture}

Please continue reading about Kafka Architecture. The next article covers [Kafka Topics Architecture](http://cloudurable.com/blog/kafka-architecture-topics/index.html) with a discussion of how partitions are used for fail-over and parallel processing.

