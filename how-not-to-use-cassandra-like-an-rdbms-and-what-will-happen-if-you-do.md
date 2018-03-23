# How Not To Use Cassandra Like An RDBMS \(and what will happen if you do\)

Cassandra isn’t a relational database management system, but it has some features that make it look a bit like one. Chief among these is CQL, a query language with an SQL-like syntax. CQL isn’t a bad thing in itself – in fact it’s very convenient – but it can be misleading since it gives developers the illusion that they are working with a familiar data model, when things are really very different under the hood. Not only is Cassandra_not_an RDBMS, it’s not even_like_an RDBMS in some of the ways you might expect. In this post I’ll review a few example scenarios where a beginner might be unpleasantly surprised by the differences, and suggest some remedies.

## Example 1: querying by non-key columns

Here’s some CQL to create a “shopping trolley contents” table in Cassandra:

```
CREATE TABLE shoppingTrolleyContents (
 trolleyId timeuuid,
 lineItemId timeuuid,
 itemId text,
 qty int,
 unitPrice decimal,
 PRIMARY KEY(trolleyId, lineItemId)
) WITH CLUSTERING ORDER BY (lineItemId ASC);
```

Suppose we wanted to find all of the shopping trolleys that contains a particular item, identified by`itemId`. Here’s a query that would make perfect sense in SQL, that Cassandra will not allow you to perform:

```
// Find all the trolleys that contain a particular item
SELECT trolleyId FROM shoppingTrolleyContents WHERE itemId = 'SKU001';
```

If you try to run this query through CQLSH, you will get the response “No secondary indexes on the restricted columns support the provided operators”. What this means is that the columns in this table are indexed only by the columns listed in the`PRIMARY KEY`clause. Without defining a “secondary index” there is no way to search for values using a`WHERE`clause restricting on any other column.

_OK_, you may think,_so I have to define secondary indexes explicitly_. A quick[Google search](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/create_index_r.html)later, and you have:

```
CREATE INDEX trolleyByItemId ON shoppingTrolleyContents (itemId);
```

You can now run the query you wanted. Unfortunately, while restrictions on primary indexes allow queries to be routed efficiently to the specific nodes in the Cassandra cluster that hold the data you’re looking for, the secondary index must be consulted on every node of the Cassandra cluster in order to find matching records. Use of secondary indexes can thus severely hamper the scalability of Cassandra, especially if the indexed column has a high cardinality \(i.e. can contain many distinct values\).

In this scenario, it is probably better to have a second table that explicitly provides for`trolleyId`lookups by`itemId`:

```
CREATE TABLE lineItemsByItemId (
 itemId text,
 trolleyId timeuuid,
 lineItemId timeuuid,
 PRIMARY KEY(itemId, trolleyId, lineItemId)
) WITH CLUSTERING ORDER BY (trolleyId DESC, lineItemId ASC);
```

However, you now have to consider the overhead of maintaining this table, and the fact that Cassandra does not support transactional updates to multiple tables with ACID guarantees.

Suppose that a line item is removed from a trolley, and a user queries for orders containing line items with that`itemId`before the`lineItemsByItemId`table has been updated. The returned`trolleyId`/`lineItemId`pair will then refer to a record in the`shoppingTrolleyContents`table that no longer exists. This is precisely the scenario that the referential integrity mechanisms \(foreign key constraints\) provided by an RDBMS are designed to avoid – but Cassandra provides_no_mechanisms for ensuring referential integrity between tables.

## Example 2: joining tables

Suppose we search in`lineItemsByItemId`for all of the line items with`itemId='SKU0001'`:

```
SELECT trolleyId, lineItemId FROM lineItemsByItemId WHERE itemId='SKU0001';
```

Because Cassandra doesn’t support`JOIN`s, if we get back 1,000`trolleyId`/`lineItemId`pairs, we must then retrieve each line item’s details separately from the`shoppingTrolleyContents`table in order to obtain the associated quantity and price data. This is tremendously inefficient. It may be better to denormalise further, copying all of the relevant columns from`shoppingTrolleyContents`into`lineItemsByItemId`:

```
CREATE TABLE lineItemsByItemId (
 itemId text,
 trolleyId timeuuid,
 lineItemId timeuuid,
 qty int,
 unitPrice decimal,
 PRIMARY KEY(itemId, trolleyId, lineItemId)
) WITH CLUSTERING ORDER BY (trolleyId DESC, lineItemId ASC);
```

This kind of duplication of data is anathema in relational database design. In the Cassandra world, it’s commonplace and necessary in order to support efficient querying.

## Example 3: reverse lookups

A shopping trolley usually has an owner, and perhaps some other metadata associated with it. Here’s the CQL for a table which associates user ids with shopping trolley ids:

```
CREATE TABLE shoppingTrolleyOwners (
 trolleyId timeuuid,
 ownerId text,
 PRIMARY KEY (trolleyId, ownerId)
) WITH CLUSTERING ORDER BY (ownerId ASC);
```

We can use this to find out who owns a given trolley by id:

```
SELECT ownerID FROM shoppingTrolleyOwners WHERE trolleyId='';
```

However, the reverse lookup, to find trolleys by owner, is not supported, even though ownerId is a primary index key:

```
SELECT trolleyId FROM shoppingTrolleyOwners WHERE ownerId='bob';
```

The error we get back if we try to run the above is:

```
Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. 
If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING.
```

Once again, if Cassandra doesn’t immediately allow you to do something, it’s worth making sure you understand why before just going ahead and adding`ALLOW FILTERING`to the query to “make it work”.

The problem here is that the`trolleyId`is used as the_partition_key, which determines which node in the Cassandra cluster the data is stored on, while the`ownerId`is used as the_cluster_key, which indexes data within the partition. In order to perform the query as written, Cassandra would have to send it to all of the nodes within the cluster, obtain the records for all`trolleyId`values within each partition, and then filter them by`ownerId`. This has the potential to be very inefficient.

A better solution, in this case, would be to create a second table to support the reverse lookup:

```
CREATE TABLE shoppingTrolleysByOwner (
 ownerId text,
 trolleyId timeuuid,
 PRIMARY KEY (ownerId, trolleyId)
) WITH CLUSTERING ORDER BY (trolleyId DESC);
```

Once again, the previous warnings about data consistency apply: both tables must be written to whenever a shopping trolley is created, and there are no ACID transactions to ensure that this pair of updates is carried out atomically.

## Conclusions

Cassandra can usefully be thought of as something like a distributed version of the storage engine that an RDBMS might use, but without the data management capabilities that an RDBMS brings in addition to the ability to persist and retrieve data.

It is certainly possible to build an RDBMS-like system on top of Cassandra, but doing so will progressively negate all of the performance and scalability advantages that Cassandra’s distributed data model provides. If you find yourself doing this – implementing in-memory joins, distributed locks or other mechanisms to get RDBMS-like behaviour out of a Cassandra-backed system – then it may be time to stop and reconsider. Will Cassandra still deliver the performance you want, if you use it in this way?

An RDBMS provides a consistent, principled approach to data management that works for all cases, at the cost of limited scalability: you pay the “relational tax” in exchange for a highly flexible yet predictable data model, By contrast, Cassandra’s is a “pay for what you use” model, in which data integrity is an application-level concern, and specific query patterns must be explicitly supported through decisions made about the physical layout of data across the cluster. It “unsolves” some of the problems the RDBMS was invented to solve, in order to address different problems more effectively.

For many purposes, a dual or polyglot approach is worth considering. Use Cassandra as the primary data store for capturing information as it arrives into the system; but then build “query-optimised views” of subsets of that data in other databases specialised to the kinds of access users require. Use an indexing engine such as SOLR to provide full-text search across records, or a graph database such as Neo4j to store metadata in a way that supports “traversal-heavy” queries that join together many different kinds of entity. Use an RDBMS when you need the full power and flexibility of SQL to express ad hoc queries that explore the data in multiple dimensions.

Alternatively, a stream-processing approach may be the way to go when you need to generate complex reports which consume large amounts of captured data. Stream the records you have captured in Cassandra into an Apache Spark cluster, and use Spark SQL to execute complex queries using filters, joins and aggregations. This approach, which I call “write first, reason later”, is especially suited to event-driven systems where capture of event information can be decoupled from decision-making about how to react to events in the aggregate.



