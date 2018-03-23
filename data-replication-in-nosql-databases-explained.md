# Data Replication in NoSQL Databases Explained

Data replication is the concept of having data, within a system, be geo-distributed; preferably through a non-interactive, reliable process. In traditional RDBMS databases, implementing any sort of replication is a struggle because these systems were not developed with horizontal scaling in mind. Instead, these systems can be backed up via a semi-manual process where live recovery wouldn’t be  much of an issue.   Even with live recovery not being much of an issue, it downplays the complexity of this setup. When dealing with today’s globally distributed data, the former colocated replication concepts will not suffice when implemented at geographic scale.

Today’s infrastructure requires systems that  natively support active and real-time replication, achieved through transparent and simple configurations. The ability to dictate where and how your data is replicated via easily tunable settings, along with providing users with easily understood concepts is what modern day NoSQL databases strive to offer.

Apache Cassandra, built with native multi data center replication in mind, is  one of the most overlooked because this level of infrastructure has been assimilated as “tribal knowledge” within the Cassandra community. For those new to Apache Cassandra, this page is meant to highlight the simple inner workings of how Cassandra excels in multi data center replication by simplifying the problem at a single-node level.

This page covers the fundamentals of Cassandra internals, multi-data center use cases, and a few caveats to keep in mind when expanding your cluster.

![](https://academy.datastax.com/sites/default/files/illustration_1-3.png)

## Handling Writes at the Node Level

Apache Cassandra was designed with horizontal scaling in mind from its infancy, at a time just before “Big Data” was becoming mainstream. This is important because up until this point in database history, horizontal scaling typically seemed to be an afterthought. Some legacy databases claim to support scaling, yet require extra infrastructures to support doing so. Some newer databases even go as far as claiming that faults across distributed systems never occur.

Our hope is that those reading this have never had to deal with these hack-based infrastructures or have never learned firsthand that faults in a distributed setting are very real and failures of minute chance will always occur at scale. However, pretending a majority of you are new to all of this would be ignorance on our part. And if we exclaim that Cassandra has solved these issues without presenting how this actually happens we may as well be selling snake oil.

That’s why, before we continue any further, we’ll cover how writes occur in Cassandra, to set the basis for understanding our claim that Cassandra is a highly available, fault tolerant, distributed, redundant database.

A mutation is any write request that hits the Cassandra servers. Write mutations, or writes, can be an insert, update, or delete command since Cassandra deals with them in the same fashion in order to elegantly handle mutations in a distributed setting. When a mutation is received two types of actions happen: synchronous actions and asynchronous actions.



### Synchronous Actions of Mutations

Synchronous actions, in this case, are typically actions that occur between receiving a request and issuing a response \[1\]. When requests come in, they are routed to both memory for present access and disk for recovery.

In memory, they are stored in memtables that exist per column family. There, rows can be overwritten or appended easily with new data and provide fast read access. But because memory clears when nodes are power cycled, all requests also are written to disk for a persistent backup.

When requests hit disk, they are stored on the Commit Log, an append-only file structure that accepts all requests as they are received. This append-only access ensures that rotational media is utilized resourcefully and solid state drives are not subject to extensive disk wear.

For highly critical systems, you can even tune the [commitlog\_sync setting](http://www.datastax.com/documentation/cassandra/2.0/cassandra/configuration/configCassandra_yaml_r.html?scroll=reference_ds_qfg_n1r_1k__commitlog_sync) to wait on acknowledging writes until the data is synchronized to disk; in most cases however, this is overkill. Writing at a consistency level of QUORUM or higher will handle most use cases adequately, while also providing lower latency writes + reads, due to the level of available replicas at write time.

If, and only if, the above tasks have been completed, will the request receive a successful response to the write. This means that at response time the node not only has a copy of the newest value in memory, but contains a hard copy in the event of power loss. In such an event, Cassandra will read all data into memory from the commit log to restore itself to the previous state before being marked as UP to the rest of the cluster.



### Asynchronous Actions of Mutations

Until this point, we’ve been talking about handling data at the local level on memory and on disk to ensure data integrity. The optionally asynchronous actions that occur on the coordinator nodes is where the Cassandra gets its multi-data center leverage from.

Before continuing, we’ll define the importance and election of being a “coordinator node” in Cassandra. Being a coordinator is something that occurs on randomly selected peers for the life of each request. Because Cassandra doesn’t have a master/slave architecture all peers are capable of receiving requests and it’s during this phase that peers become coordinators and synchronously ensure Consistency Levels are met. Their asynchronous responsibilities are to ensure that additional replicas are also delivered any received mutations.

It’s during this phase that mutations are bundled up and sent to nodes currently accepting writes. In the case that nodes are not available, the coordinator leaves hints on the local machine to be delivered once the machine becomes available.

This phase can occur asynchronously because Cassandra has the advantage of tunable consistency. If there are ever requests that require higher consistency, increasing write and/or read consistencies can overlook these asynchronous mutations by requiring agreement from more nodes, but in most cases applications can be designed to accept and appropriately handle eventual consistency.

![](https://academy.datastax.com/sites/default/files/illustration_2-32.png)

## Multi-Data Center Awareness at Little Extra Cost

Because Cassandra has been designed to work well at the peer-to-peer level with eventual consistency, we can take the above concepts and have multi-data center awareness with very little additional logic.

Most times data centers remain in semi-isolation, transferring data back and forth as applications connect with only the closest \(“local” as opposed to “remote”\) data center. This facilitates the workload separation, live backup, and geographic location scenarios covered in the next sections. However, occasionally an application may want to use a data center in a more active role by using a [Consistency Level of EACH\_QUORUM](http://www.datastax.com/documentation/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html).

When the EACH\_QUORUM Consistency Level is used, the coordinator acts roughly in the same fashion as above, yet knowingly sends a request to a remote data center to have a remote coordinator node handle that request locally. Once that remote coordinator gets a LOCAL\_QUORUM-type response, it responds back to the original coordinator, who resolves any conflicts, if applicable, before responding to the application request with an EACH\_QUORUM-level response.



## Real World Use Cases of Multiple Data Centers

Hopefully it has become apparent that Cassandra excels at being a distributed database at the core level and as such allows for almost free multi-data center awareness. Now that you understand how Cassandra’s multi-data center approach works and why you can trust it, do take a minute to realize how this affects your application when you move into new data centers: typically it doesn’t.

Cassandra houses all this logic internally on each of the nodes in the cluster and by doing so allows applications to function independently of the Cassandra topology. The only caveat to watch out for is properly using Consistency Levels and Replication Strategies with scaling in mind… that’s it!

When the time comes to implement the upcoming scenarios, double check Consistency Levels and Replication Strategies, simply bootstrap a few nodes into new data centers, and properly set the new Replication Strategies. Continue focusing on your application logic as Cassandra handles the high availability without your worrying about data integrity or actively maintaining multiple data centers!



### Workload Separation

By implementing data centers as the divisions between varying workloads, DataStax Enterprise allows a natural distribution of data from real-time data centers to near real-time analytics and search data centers.

Whenever a write comes in via a client application, it hits the main Cassandra data center and returns the acknowledgment at the current consistency level \(typically less than EACH\_QUORUM, to allow for a high throughput and low latency\). In parallel and asynchronously, these writes are sent off to the Analytics and Search data centers based on the replication strategy for active keyspaces. A typical replication strategy would look similar to {Cassandra: 3, Analytics: 2, Search: 1}, depending on use cases and throughput requirements.

Once these asynchronous hints are received on the additional clusters, they undergo the normal write procedures and are assimilated into that data center. This way, any analytical jobs that are running can easily and simply access this new data without a time-consuming ETL process. For DataStax Enterprises Search nodes, these writes are introduced into the memtables and additional Search processes are triggered to incorporate this data.

Don’t assume this can only be done on DataStax Enterprise due to proprietary code. DataStax Enterprise delivers a complete Analytics and Search package, but any Apache Cassandra installation, that is separated by data centers, can benefit from the same workflow. Simply set up one data center to be the real-time data center that’s accessed by your application layer and another data center to house intensive workloads. Then your separate data center can use near-real time data to churn through computations in scratch column families before writing results out to shared analytical column families. Once these results are written out locally to the shared column families, they’ll soon appear in the real-time data center ready for live usage.



### Live Backup Scenario

Some Cassandra use cases instead use different data centers as a live backup that can quickly be used as a fallback cluster. We will cover the most common use case using Amazon’s Web Services Elastic Cloud Computing \(EC2\) in the following example.

#### Specific Use Case

Allow your application to have multiple fallback patterns across multiple consistencies and data centers.

#### Example

A client application was created and currently sends requests to EC2′s US-East-1 region at a Consistency Level of QUORUM. Later, another data center is added to EC2′s US-West-1 region to serve as a live backup. The replication strategy can be a full live backup \({US-East-1: 3, US-West-1: 3}\) or a smaller live backup \({US-East-1: 3, US-West-1: 2}\) to save on cost and disk usage for this regional outage scenario. All clients continue to write to the US-East-1 nodes by ensuring that the client’s load balancing pools are restricted to just those nodes, to minimize cross data center latency.

To implement a better cascading fallback, initially the client’s connection pool will only be aware of all nodes in the US-East-1 region. In the event of client errors, all requests will retry at a Consistency Level of QUORUM, for X times, then decrease to a Consistency Level of ONE while escalating the appropriate notifications. If the requests are still unsuccessful, using a new connection pool consisting of nodes from the US-West-1 data center, requests should begin contacting US-West-1 at a higher Consistency Level, before ultimately dropping down to a Consistency Level of ONE.

For cases like this, natural events and other failures can be prevented from affecting your live applications.



### Live Recovery

As long as the original data center is restored within gc\_grace\_seconds \(10 days by default\), perform a rolling repair \(without the -pr option\) on all of its nodes once they come back online. Doing so will ensure that all data is consistent and that all deletes have properly been applied.

If, however, the nodes will be set to come up and complete the repair commands after gc\_grace\_seconds, you will need to take the following steps to ensure that deleted records are not reinstated:

* ensure your applications are only contacting the backup data center

* remove all the offending nodes from the ring using nodetool removetoken

* clear all data from the failed region \(data directories, commit logs, snapshots, and system logs\)

* decrement all tokens in this region if not using vnodes

* disable auto\_bootstrap

* start up the entire failed region

* and, finally, run a rolling repair \(without the -pr option\) on all nodes in the recovered region

After these nodes are up to date, you can restart your applications and continue using your primary data center.



### Geographic Location Scenario

There are certain use cases where data should be housed in different data centers depending on a user’s location in order to provide a more responsive exchange. The use case we will cover refers to data centers in different countries, but the same logic and procedures apply for data centers in different regions. The logic that defines which data center a user will be connected to resides at the application level.

#### Specific Use Case

Have users connect to data centers based on geographic location, but ensure this data is available cluster-wide for backup, analytics, and to account for user travel across regions.



#### Example

The required end result is for users in the US to contact one data center while UK users contact another to lower end-user latency. An additional requirement is for both of these data centers to be a part of the same cluster to lower operational costs and facilitate cross-data center replication. This can be handled using the following rules:

* When reading and writing from each data center, ensure that the client’s load balancing policies prefers the local data center

* If doing reads at QUORUM, ensure that LOCAL\_QUORUM is being used and not EACH\_QUORUM since this latency will affect the end user’s performance experience due to cross-data center resolutions

* Depending on how consistent you want your data centers to be, you may choose to run repair operations \(without the -pr option\) more frequently than the required once per gc\_grace\_seconds



#### Benefits

When setting up clusters in this manner:

* You ensure faster performance for each end user

* You gain multi-region live backups for free, just as mentioned in the section above. Granted, the performance for requests across the US and UK will not be as fast, but your application does not have to hit a complete standstill in the event of catastrophic losses

* Users can travel across regions and in the time taken to travel, the user’s information should have finished replicating asynchronously across regions

## 

## Caveats of Multi Data Center Replication

* Always use the NetworkTopologyStrategy when using, or planning to use, multiple data centers. SimpleStrategy will work across data centers, but has the following disadvantages:

  * Does not provide any of the features mentioned above

  * Introduces latency on each write \(depending on data center distance and latency between data centers\)

* Defining one rack for the entire cluster is the simplest and most common implementation. Multiple racks should be avoided for the following reasons:

  * Most users tend to ignore or forget rack requirements that state racks should be in an alternating order to allow the data to get distributed safely and appropriately

  * Many users are not using the rack information effectively by using a setup with as many racks as they have nodes, or similar non-beneficial scenarios

  * When using racks correctly, each rack should typically have the same number of nodes

  * In a scenario that requires a cluster expansion while using racks, the expansion procedure can be tedious since it typically involves several node moves and has to ensure that racks will be distributing data correctly and evenly. At times when clusters need immediate expansion, racks should be the last things to worry about

**Notes**

1. Responses can be issued without validation by using the non-default [Consistency Level of ANY](http://www.datastax.com/documentation/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html?scroll=concept_ds_umf_5xx_zj__about-write-consistency), but we ignore this anti-pattern for simplicity’s sake.

