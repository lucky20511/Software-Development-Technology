# Understand replication {#understand-replication}

## How is data replicated among nodes? {#how-is-data-replicated-among-nodes}

SimpleStrategy: create replicas on nodes subsequent  
to the primary range node![](http://marianogappa.github.io/cassandra-replication-slides/images/ring.png)

## How is data replicated between data centers? {#how-is-data-replicated-between-data-centers}

NetworkTopologyStrategy: distribute replicas across racks and data centers![](http://image.slidesharecdn.com/goingnativewithapachecassandraslideshare-140413024401-phpapp01/95/going-native-with-apache-cassandra-6-638.jpg?cb=1397388490)

## What is a hinted handoff? {#what-is-a-hinted-handoff}

A recovery mechanism for writes targeting offline nodes. Coordinator can store a hinted handoff if target node for a write

* is known to be down, or
* fails to acknowledge

Coordinator stores the hint in its`system.hints`table. The write is replayed when the target node comes online

