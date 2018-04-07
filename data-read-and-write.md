# How is data read {#how-is-data-read}

For a read request, Cassandra consults an in-memory data structure called a Bloom filter that checks the probability of an SSTable having the needed data. The Bloom filter can tell very quickly whether the file probably has the needed data, or certainly does not have it. If answer is a tenative yes, Cassandra consults another layer of in-memory caches, then fetches the compressed data on disk. If the answer is no, Cassandra doesn't trouble with reading that SSTable at all, and moves on to the next.

To satisfy a read, Cassandra must combine results from the active memtable and potentially multiple SSTables. Cassandra processes data at several stages on the read path to discover where the data is stored, starting with the data in the memtable and finishing with SSTables:

* Check the memtable
* Check row cache, if enabled
* Checks Bloom filter
* Checks partition key cache, if enabled
* Goes directly to the compression offset map if a partition key is found in the partition key cache, or checks the partition summary if not If the partition summary is checked, then the partition index is accessed
* Locates the data on disk using the compression offset map
* Fetches the data from the SSTable on disk

![](https://academy.datastax.com/sites/default/files/read-path.png "Figure 3. The Cassandra Read Path")



# How is data written {#how-is-data-written}

Data written to a Cassandra node is first recorded in an on-disk commit log and then written to a memory-based structure called a memtable. When a memtable's size exceeds a configurable threshold, the data is written to an immutable file on disk called an SSTable. Buffering writes in memory in this way allows writes always to be a fully sequential operation, with many megabytes of disk I/O happening at the same time, rather than one at a time over a long period. This architecture gives Cassandra its legendary write performance.

Cassandra processes data at several stages on the write path, starting with the immediate logging of a write and ending in with a write of data to disk:

* Logging data in the commit log
* Writing data to the memtable
* Flushing data from the memtable
* Storing data on disk in SSTables

![](https://academy.datastax.com/sites/default/files/write-path.png "Figure 2. The Cassandra Write Path")

