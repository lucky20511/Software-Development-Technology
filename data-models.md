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

![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen%20Shot%202016-02-24%20at%2011.46.09.png)

Rows may be described as`skinny`or`wide`

* Skinny row: has a fixed, relatively small number of column keys
* Wide row: has a relatively large number of column keys \(hundreds or thousands\); this number may increase as new data values are inserted

## Key \(Partition Key\) {#key-partition-key}

### Composite row key {#composite-row-key}

multiple components separated by colon![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen%20Shot%202016-02-24%20at%2011.53.17.png)

### Composite column key {#composite-column-key}

multiple components separated by colon![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen%20Shot%202016-02-24%20at%2011.54.01.png)

## Column family \(Table\) {#column-family-table}

set of rows with a similar structure  
![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen%20Shot%202016-02-24%20at%2011.56.28.png)

## Table with single-row partitions {#table-with-singlerow-partitions}

![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen%20Shot%202016-02-24%20at%2012.24.12.png)

## Table with multi-row partitions {#table-with-multirow-partitions}

![](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen%20Shot%202016-02-24%20at%2012.25.54.png)

