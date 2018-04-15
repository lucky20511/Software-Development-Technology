# Why Running Spark on Cassandra

CQL of Cassandra is very, very basic. It have basic aggregation functions added in latest versions, but Cassandra wasn't designed for analytical workloads, and probably you will both struggle to run analytical queries and will "kill" your cluster performance.

So what Spark is trying to solve there, is executing analytical queries on top of data that sitting in Cassandra. Using Spark, you can take data from many sources \(RDBMS, files, hadoop etc.\) and execute analytical queries versus that data.





  


