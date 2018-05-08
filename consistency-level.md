# Understand and tune consistency {#understand-and-tune-consistency}

## What is consistency? {#what-is-consistency}

Consistency Level: sets how many of the nodes to be sent a given request must acknowledge that request, for a response to be returned to the client

* Write request: how many nodes must acknowledge they received and wrote the write request?
* Read request: how many nodes must acknowledge by sending their most recent copy of the data?

## Write consistency levels {#write-consistency-levels}

| Level | Description | Usage |
| :--- | :--- | :--- |
| ALL | A write must be written to the [commit log and memtable](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage) on all replica nodes in the cluster for that partition. | Provides the highest consistency and the lowest availability of any other level. |
| EACH\_QUORUM | Strong consistency. A write must be written to the [commit log and memtable](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage) on a quorum of replica nodes in _all _[data centers](http://docs.datastax.com/en/cassandra/2.1/share/glossary/gloss_data_center.html). | Used in multiple data center clusters to strictly maintain consistency at the same level in each data center. For example, choose this level if you want a read to fail when a data center is down and the QUORUM cannot be reached on that data center. |
| QUORUM | A write must be written to the [commit log and memtable](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage) on a quorum of replica nodes. | Provides strong consistency if you can tolerate some level of failure. |
| LOCAL\_QUORUM | Strong consistency. A write must be written to the[commit log and memtable](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage)on a quorum of replica nodes in the same data center as the[coordinator node](http://docs.datastax.com/en/cassandra/2.1/share/glossary/gloss_coordinator_node.html). Avoids latency of inter-data center communication. | Used in multiple data center clusters with a rack-aware replica placement strategy, such as[NetworkTopologyStrategy](http://docs.datastax.com/en/cassandra/2.1/cassandra/architecture/architectureDataDistributeReplication_c.html), and a properly configured snitch. Use to maintain consistency locally \(within the single data center\). Can be used with[SimpleStrategy](http://docs.datastax.com/en/cassandra/2.1/cassandra/architecture/architectureDataDistributeReplication_c.html). |
| ONE | A write must be written to the[commit log and memtable](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage)of at least one replica node. | Satisfies the needs of most users because consistency requirements are not stringent. |
| TWO | A write must be written to the[commit log and memtable](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage)of at least two replica nodes. | Similar to ONE. |
| THREE | A write must be written to the[commit log and memtable](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage)of at least three replica nodes. | Similar to TWO. |
| LOCAL\_ONE | A write must be sent to, and successfully acknowledged by, at least one replica node in the local datacenter. | In a multiple data center clusters, a consistency level of ONE is often desirable, but cross-DC traffic is not. LOCAL\_ONE accomplishes this. For security and quality reasons, you can use this consistency level in an offline datacenter to prevent automatic connection to online nodes in other data centers if an offline node goes down. |
| ANY | A write must be written to at least one node. If all replica nodes for the given partition key are down, the write can still succeed after a[hinted handoff](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_about_hh_c.html)has been written. If all replica nodes are down at write time, an ANY write is not readable until the replica nodes for that partition have recovered. | Provides low latency and a guarantee that a write never fails. Delivers the lowest consistency and highest availability. |
| SERIAL | Achieves [linearizable consistency](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_tunable_consistency_c.html) for lightweight transactions by preventing unconditional updates. | You cannot configure this level as a normal consistency level, configured at the driver level using the consistency level field. You configure this level using the serial consistency field as part of the[native protocol operation](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/en/developer/java-driver/2.1/java-driver/reference/queryBuilderOverview.html). See failure scenarios. |
| LOCAL\_SERIAL | Same as SERIAL but confined to the data center. A write must be written conditionally to the[commit log and memtable](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage)on a quorum of replica nodes in the same data center. | Same as SERIAL. Used for disaster recovery. See failure scenarios. |

## Read consistency levels {#read-consistency-levels}

| Level | Description | Usage |
| :--- | :--- | :--- |
| ALL | Returns the record after all replicas have responded. The read operation will fail if a replica does not respond. | Provides the highest consistency of all levels and the lowest availability of all levels. |
| EACH\_QUORUM | Not supported for reads. | Not supported for reads. |
| QUORUM | Returns the record after a quorum of replicas has responded from any[data center](http://docs.datastax.com/en/cassandra/2.1/share/glossary/gloss_data_center.html). | Ensures strong consistency if you can tolerate some level of failure. |
| LOCAL\_QUORUM | Returns the record after a quorum of replicas in the current data center as the [coordinator node](http://docs.datastax.com/en/cassandra/2.1/share/glossary/gloss_coordinator_node.html) has reported. Avoids latency of inter-data center communication. | Used in multiple data center clusters with a rack-aware replica placement strategy \( NetworkTopologyStrategy\) and a properly configured snitch. Fails when using SimpleStrategy. |
| ONE | Returns a response from the closest replica, as determined by the [snitch](http://docs.datastax.com/en/cassandra/2.1/cassandra/architecture/architectureSnitchesAbout_c.html). By default, a [read repair](http://docs.datastax.com/en/cassandra/2.1/share/glossary/gloss_read_repair.html) runs in the background to make the other replicas consistent. | Provides the highest availability of all the levels if you can tolerate a comparatively high probability of stale data being read. The replicas contacted for reads may not always have the most recent write. |
| TWO | Returns the most recent data from two of the closest replicas. | Similar to ONE. |
| THREE | Returns the most recent data from three of the closest replicas. | Similar to TWO. |
| LOCAL\_ONE | Returns a response from the closest replica in the local data center. | Same usage as described in the table about write consistency levels. |
| SERIAL | Allows reading the current \(and possibly [uncommitted](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html#concept_ds_wt3_32w_zj__logging-writes-and-memtable-storage)\) state of data without proposing a new addition or update. If a SERIAL read finds an uncommitted transaction in progress, it will commit the transaction as part of the read. Similar to QUORUM. | To read the latest value of a column after a user has invoked a[lightweight transaction](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_ltwt_transaction_c.html)to write to the column, use SERIAL. Cassandra then checks the inflight lightweight transaction for updates and, if found, returns the latest data. |
| LOCAL\_SERIAL | Same as SERIAL, but confined to the data center. Similar to LOCAL\_QUORUM. | Used to achieve[linearizable consistency](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_tunable_consistency_c.html)for lightweight transactions. |

## About the QUORUM levels {#about-the-quorum-levels}

$$quorum = (sum of replication\_factors / 2) + 1$$

The sum of all the replication\_factor settings for each data center is the sum of replication factors.

If consistency is a top priority, you can ensure that a read always reflects the most recent write by using the following formula:

$$(nodes written + nodes read) > replication factor $$

