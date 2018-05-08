# Partition vs Replication

[https://dzone.com/articles/kafka-topic-architecture-replication-failover-and](https://dzone.com/articles/kafka-topic-architecture-replication-failover-and)

[https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk\_kafka-component-guide/content/ch\_create-kafka-topic.html](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk_kafka-component-guide/content/ch_create-kafka-topic.html)

Youtube:

[https://www.youtube.com/watch?v=udnX21\_\_SuU](https://www.youtube.com/watch?v=udnX21__SuU)

### Produce Record

ProduceRecord constructor contains **Topic**, **Key** and **Value**.

| `send(ProducerRecord<K,V> record):Future<RecordMetadata>;` |
| :--- |


A`ProducerRecord`contains a key \(K\) and a value \(V\). The value is the payload of your message \(unlike in JMS there is no`Message`construct that allows you to send additional message metadata\). The key is a business value provided by you that is used to shard your messages across the partitions.

