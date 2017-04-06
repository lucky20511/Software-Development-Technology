## Step 1: Use cases and Constraints

Scope the problem. Don't make assumption, ask the question to get the information of use cases\(the service you'e gonna provide\) and the constraints\(the data size you're gonna cope with or you need to store\) you might meet.

## Step 2: Abstract Design

Illustrate the basic components of system and the relation between them.

## Step 3: Bottleneck

According to the use cases, constraints and the abstract design by the step1 and step2, you can roughly point the all possible bottlenecks of this system. So you can carefully take care of them in the later design steps.

## Step 4: Scalability

Scale up vertical

Scale out horizontal



load balance Mechanism:

Round Robin

Consisten Hashing

Rendezvous



Cache

DNS could do the load balancing  but  the better way is just leave it to the real load balancer



Sticky session: \(if the user visit a same web multiple time, he will keep the session whatever.\)



The load balancing might break the session. But the cookie can work for this.

The information store in the cookie should avoid to be something might change.



