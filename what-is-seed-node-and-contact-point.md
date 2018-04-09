# Seed Node and Contact Points

Seed nodes serve two purposes.

1. they act as a place for new nodes to announce themselves to a cluster. so,
   **without at least one live seed node, no new nodes can join the cluster**
   because they have no idea how to contact non-seed nodes to get the cluster status.
2. seed nodes act as gossip hot spots. since nodes gossip more often with seeds than non-seeds, the seeds tend to have more current information, and therefore the whole cluster has more current information. this is the reason
   **you should**
   _**not**_
   **make all nodes seeds**
   . similarly, this is also why all nodes in a given data center should have the same list of seed nodes in their cassandra.yaml file. typically, 3 seed nodes per data center is ideal.

the cassandra client **contact points** simply provide the cluster topology to the client, after which the client may connect to any node in the cluster. as such, they are similar to seed nodes and it makes sense to use the same nodes for both seeds and client contacts. however,**you can safely configure as many cassandra client contact points as you like**. the only other consideration is that the first node a client contacts sets its data center affinity, so you may wish to order your contact points to prefer a given data center.

for more details about contact points see this question:[Cassandra Java driver: how many contact points is reasonable?](https://stackoverflow.com/questions/26852413/cassandra-java-driver-how-many-contact-points-is-reasonable)

