# Understand the Cassandra data model {#understand-the-cassandra-data-model}

The Cassandra data model defines

* Column family as a way to store and organize data
* Table as a two-dimensional view of a multi-dimensional column family
* Operations on tables using the Cassandra Query Language \(CQL\)

Cassandra1.2+reliesonCQLschema,concepts,andterminology, though the older Thrift API remains available

| Table \(CQL API terms\) | Column Family \(Thrift API terms\) |
| :--- | :--- |
| Table is a set of partitions | Column family is a set of rows |
| Partition may be single or multiple row | Row may be skinny or wide |
| Partition key uniquely identifies a partition, and may be simple or composite | Row key uniquely identifies a row, and may be simple or composite |
| Column uniquely identifies a cell in a partition, and may be regular or clustering | Column key uniquely identies a cell in a row, and may be simple or composite |
| Primary key is comprised of a partition key plus clustering columns, if any, and uniquely identifies a row in both its partition and table |  |

## Row \(Partition\) {#row-partition}

Row is the smallest unit that stores related data in Cassandra

* Rows: individual rows constitute a column family
* Row key: uniquely identifies a row in a column family
* Row: stores pairs of column keys and column values
* Column key: uniquely identifies a column value in a row
* Column value: stores one value or a collection of values

![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen Shot 2016-02-24 at 11.46.09.png)

Rows may be described as`skinny`or`wide`

* Skinny row: has a fixed, relatively small number of column keys
* Wide row: has a relatively large number of column keys \(hundreds or thousands\); this number may increase as new data values are inserted

##  {#key-partition-key}

## Key {#key-partition-key}

A compound primary key consists of the partition key and one or more additional[columns](https://docs.datastax.com/en/glossary/doc/glossary/gloss_column.html)that determine clustering. The[partition key](https://docs.datastax.com/en/glossary/doc/glossary/gloss_partition_key.html)determines which node stores the data. It is responsible for data distribution across the nodes. The additional columns determine per-partition clustering.[Clustering](https://docs.datastax.com/en/glossary/doc/glossary/gloss_clustering.html)is a storage engine process that sorts data within the partition.

In a simple primary key, Apache Cassandra™ uses the first column name as the partition key. \(Note that Cassandra can use

[multiple columns](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/refCompositePk.html)

in the definition of a partition key.\) In the music service's playlists table, id is the partition key. The remaining columns can be defined as

[clustering columns](https://docs.datastax.com/en/glossary/doc/glossary/gloss_clustering_column.html)

. In the playlists table below, the song\_order is defined as the clustering column column:

```
PRIMARYKEY (id, song_order));
```

The data for each partition is clustered by the remaining column or columns of the primary key definition. On a physical node, when rows for a partition key are stored in order based on the clustering columns, retrieval of rows is very efficient. For example, because the id in the playlists table is the partition key, all the songs for a playlist are clustered in the order of the remaining song\_order column. The others columns are displayed in alphabetical order by Cassandra.

Insertion, update, and deletion operations on rows sharing the same partition key for a table are performed atomically and in isolation.

You can query a single sequential set of data on disk to get the songs for a playlist.

```
SELECT * FROM playlists WHERE id = 62c36092-82a1-3a00-93d1-46196ee77204
  ORDER BY song_order DESC LIMIT 50;
```

The output looks something like this:



![](https://docs.datastax.com/en/cql/3.1/cql/images/select_desc.png)



Cassandra stores an entire row of data on a node by partition key. If you have too much data in a partition and want to spread the data over multiple nodes, use a [composite partition key](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/refCompositePk.html).

## Key \(Partition Key\) {#key-partition-key}

### Composite row key {#composite-row-key}

multiple components separated by colon![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen Shot 2016-02-24 at 11.53.17.png)

### Composite column key {#composite-column-key}

multiple components separated by colon![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen Shot 2016-02-24 at 11.54.01.png)

## Column family \(Table\) {#column-family-table}

set of rows with a similar structure  
![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen Shot 2016-02-24 at 11.56.28.png)

## Table with single-row partitions {#table-with-singlerow-partitions}

![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen Shot 2016-02-24 at 12.24.12.png)

## Table with multi-row partitions {#table-with-multirow-partitions}

![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen Shot 2016-02-24 at 12.25.54.png)

