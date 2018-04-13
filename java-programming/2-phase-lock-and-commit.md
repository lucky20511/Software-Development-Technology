# Two-phase locking {#firstHeading}

**two-phase locking**

\(**2PL**\) is a [concurrency control](https://en.wikipedia.org/wiki/Concurrency_control) method that guarantees [serializability](https://en.wikipedia.org/wiki/Serializability).

By the 2PL protocol, locks are applied and removed in two phases:

1. Expanding phase: locks are acquired and no locks are released.
2. Shrinking phase: locks are released and no locks are acquired.

Two types of locks are utilized by the basic protocol: Shared and Exclusive locks. Refinements of the basic protocol may utilize more lock types. Using locks that block processes, 2PL may be subject to [deadlocks](https://en.wikipedia.org/wiki/Deadlock) that result from the mutual blocking of two or more transactions.

# Two-phase commit {#firstHeading}

In a "normal execution" of any single[distributed transaction](https://en.wikipedia.org/wiki/Distributed_transaction)\( i.e., when no failure occurs, which is typically the most frequent situation\), the protocol consists of two phases:

1. The commit-request phase \(or voting phase\), in which a coordinator process attempts to prepare all the transaction's participating processes \(named participants, cohorts, or workers\) to take the necessary steps for either committing or aborting the transaction and to vote, either "Yes": commit \(if the transaction participant's local portion execution has ended properly\), or "No": abort \(if a problem has been detected with the local portion\), and
2. The commit phase, in which, based on voting of the cohorts, the coordinator decides whether to commit \(only if all
   have voted "Yes"\) or abort the transaction \(otherwise\), and notifies the result to all the cohorts. The cohorts then follow with the needed actions \(commit or abort\) with their local transactional resources \(also called
   recoverable resources; e.g., database data\) and their respective portions in the transaction's other output \(if applicable\).

Note that the two-phase commit \(2PC\) protocol should not be confused with the [two-phase locking](https://en.wikipedia.org/wiki/Two-phase_locking)\(2PL\) protocol, a [concurrency control](https://en.wikipedia.org/wiki/Concurrency_control) protocol..

