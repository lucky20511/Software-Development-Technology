# Understand and use the DDL subset of CQL {#understand-and-use-the-ddl-subset-of-cql}

## Keyspace {#keyspace}

a top-level namespace for a CQL table schema

* Defines the replication strategy for a set of tables
  * Keyspace per application is a good idea
* Data objects \(e.g., tables\) belong to a single keyspace

## How are primary key, partition key, and clustering columns defined? {#how-are-primary-key-partition-key-and---clustering-columns-defined}

* Simple partition key, no clustering columns

  `PRIMARY KEY ( partition_key_column )`

* Composite partition key, no clustering columns

  `PRIMARY KEY ( ( partition_key_col1, ..., partition_key_colN ) )`

* Simple partition key and clustering columns

  `PRIMARY KEY ( partition_key_column,clustering_column1, ..., clustering_columnM )`

* Composite partition key and clustering columns

  `PRIMARY KEY ( ( partition_key_col1, ..., partition_key_colN ), clustering_column1, ...,clustering_columnM ) )`

## UUID {#uuid}

universally unique identifiers, which is used for avoiding column collision.

Format:`hex{8}-hex{4}-hex{4}-hex{4}-hex{12}`

## TIMEUUID {#timeuuid}

* Embeds a time value within a UUID
* CQL function now\(\) generates a new TIMEUUID
* CQL function dateOf\(\) extracts the embedded timestamp as a date
* TIMEUUID values in clustering columns or in column names are ordered based on time

## TIMESTAMP {#timestamp}

64-bit integer representing a number of milliseconds since January 1 1970 at 00:00:00 GMT

## COUNTER {#counter}

* Cassandra supports distributed counters
* Useful for tracking a count
* Counter column stores a number that can only be updated
  * Incremented or decremented
  * Cannot assign an initial value to a counter \(initial value is 0\)
* Counter column cannot be part of a primary key
* If a table has a counter column, all non-counter columns must be part of a primary key

## What is a secondary index? {#what-is-a-secondary-index}

* Can index additional columns to enable searching by those columns
* Cannot be created for
  * counter columns
  * static columns

## When do you want to use a secondary index? {#when-do-you-want-to-use-a-secondary-index}

* Use with low-cardinality columns

  Columns that may contain a relatively small set of distinct values

Do not use:

* On high-cardinality columns
* On counter column tables
* On a frequently updated or deleted columns
* To look for a row in a large partition unless narrowly queried

# ------------------------------- {#understand-and-use-the-dml-subset-of-cql}

#  {#understand-and-use-the-dml-subset-of-cql}

# Understand and use the DML subset of CQL {#understand-and-use-the-dml-subset-of-cql}

## What is the syntax of the INSERT statement? {#what-is-the-syntax-of-the-insert-statement}

`INSERT INTO table_name (column1, column2 ...) VALUES (value1, value2 ...)`

## What is the syntax of the UPDATE statement? {#what-is-the-syntax-of-the-update-statement}

`UPDATE <keyspace>.<table> SET column_name1 = value, column_name2 = value WHERE primary_key_column = value;`

* Row must be identified by values in primary key columns

## What is an "upsert"? {#what-is-an-upsert}

* Cassandra is a distributed database that avoids reading before a write, so an INSERT or UPDATE sets the column values you specify regardless of whether the row already exists. This means inserts can update existing rows, and updates can create new rows. It also means it’s easy to accidentally overwrite existing data, so keep that in mind.

* Both UPDATE and INSERT are write operations

* No reading before writing

## What are lightweight transactions or ‘compare and set’? {#what-are-lightweight-transactions-or-compare-and-set}

### Introduces a new clause IF NOT EXISTS for inserts {#introduces-a-new-clause-if-not-exists-for-inserts}

* Insert operation executes if a row with the same primary key does not exist
* Uses a consensus algorithm called Paxos to ensure inserts are done serially
* Multiple messages are passed between coordinator and replicas with a large performance penalty
* \[applied\] column returns true if row does not exist and insert executes
* \[applied\] column is false if row exists and the existing row will be returned

### Update uses IF to verify the value for column\(s\) before execution {#update-uses-if-to-verify-the-value-for-columns-before-execution}

* \[applied\] column returns true if condition\(s\) matches and update written
* \[applied\] column is false if condition\(s\) do not match and the current row will be returned

## What is the purpose of the BATCH statement? {#what-is-the-purpose-of-the-batch-statement}

BATCH statement combines multiple INSERT, UPDATE, and DELETE statements into a single logical operation

* Saves on client-server and coordinator-replica communication
* Atomic operation
  * If any statement in the batch succeeds, all will
* No batch isolation
  * Other “transactions” can read and write data being affected by a partially executed batch

Example:

```
BEGIN BATCH
  DELETE FROM albums_by_performer WHERE performer = 'The Beatles' AND year = 1966 AND title = 'Revolver'; 
  INSERT INTO albums_by_performer (performer, year, title, genre) VALUES ('The Beatles', 1966, 'Revolver', 'Rock');
APPLY BATCH;
```

Lightweight transactions in batch

* Batch will execute only if conditions for all lightweight transactions are met
* All operations in batch will execute serially with the increased performance overhead

Example:

```
BATCH
  UPDATE user SET lock = true IF lock = false; WHERE performer = 'The Beatles' AND year = 1966 AND title = 'Revolver'; 
  INSERT INTO albums_by_performer (performer, year, title, genre) VALUES ('The Beatles', 1966, 'Revolver', 'Rock');
  UPDATE user SET lock = false; 
APPLY BATCH;
```



