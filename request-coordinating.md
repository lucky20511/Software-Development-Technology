# Understand how requests are coordinated {#understand-how-requests-are-coordinated}

https://www.slideshare.net/DataStax/understanding-data-partitioning-and-replication-in-apache-cassandra

Cassandra has a masterless 「ring」 architecture that is elegant, easy to set up, and easy to maintain.

![](https://academy.datastax.com/sites/default/files/cassandra-ring_0.png "Figure 1. Cassandra sports a masterless ring architecture")

## What is a cluster? {#what-is-a-cluster}

* Node: one Cassandra instance
* Rack: a logical set of nodes
* Data Center: a logical set of racks
* Cluster: the full set of nodes which map to a single complete token ring
  ![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen Shot 2016-02-23 at 15.24.40.png)

## What is a coordinator? {#what-is-a-coordinator}

The node chosen by the client to receive a particular read or write request to its cluster

* Any node can coordinate any request
* Each client request may be coordinated by a a different node
* The coordinator manages the Replication Factor \(RF\)
  * Replication factor \(RF\) – onto how many nodes should a write be copied?
* The coordinator also applies the Consistency Level \(CL\)

  * Consistency level \(CL\) – how many nodes must acknowledge a read or write request

    ![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen Shot 2016-02-23 at 15.png)

## What is the partitioner? {#what-is-the-partitioner}

A system on each node which hashes tokens from designated values in rows being added

## How does a partitioner work? {#how-does-a-partitioner-work}

A node's partitioner hashes a token from the partition key value of a write request

## What partitioners does Cassandra offer? {#what-partitioners-does-cassandra-offer}

Cassandra offers three partitioners

* Murmur3Partitioner \(default\): uniform distribution based on Murmur3 hash
* RandomPartitioner: uniform distribution based on MD5 hash
* ByteOrderedPartitioner \(legacy only\): lexical distribution based on key bytes

## What are virtual nodes? {#what-are-virtual-nodes}

There's one token per node, and thusly a node owns exactly one contiguous range in the ringspace.Vnodes change this paradigm from one token or range per node, to many per node. Within a cluster these can be randomly selected and be non-contiguous, giving us many smaller ranges that belong to each node.![](http://www.datastax.com/wp-content/uploads/2012/10/VNodes1.png)

## How are virtual nodes helpful? {#how-are-virtual-nodes-helpful}

* token ranges are distributed, so machines bootstrap faster
* impact of virtual node failure is spread across entire cluster
* token range assignment automated



