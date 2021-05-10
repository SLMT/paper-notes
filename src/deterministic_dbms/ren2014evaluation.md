# VLDB'14 - An evaluation of the advantages and disadvantages of deterministic database systems

- Authors: Kun Ren, Alexander Thomson, Daniel J. Abadi
- Institute: Northestern Polytechnical University, Yale University
- Published at VLDB'14
- Paper Link: <https://dl.acm.org/doi/10.14778/2732951.2732955>

## Goal

To evaluate and compare deterministic DBMSs and non-deterministic DBMSs in different settings and workloads, in order to find out where to use deterministic DBMSs is the best.

## Implementation Details

### Deterministic DBMSs

- Use VLL protocol by default
- 1 thread for acquiring locks and 4 threads for processing transactions

### Non-deterministic DBMSs

- 5 threads are used for processing transactions
  - A thread can process multiple transactions at once, if most of transactions are waiting for network messages
- Lock-based protocol
  - Use wait-for graph to detect distributed deadlocks
- Uses two phase commit to ensure strong consistency


## Key Observations

- Lock acquisition time for each transaction in deterministic DBMSs
  - 30% for short transactions (1 read/write action for an item)
  - 16% for long transactions
- VLL protocol is useful only when lock acquisition is a bottleneck.
- Two phase commit makes a non-deterministic DBMS perform poorly when there are many distributed transactions.
  - About 30% in an extreme case.
- Distributed deadlocks makes a non-deterministic DBMS perform poorly when both the number of distributed transactions and the contention are high. (Figure 1, Figure 2)
- How many nodes involve in a distributed transaction does not affect the performance difference between determinisitic and non-deterministic DBMSs. (Figure 3)
- A non-deterministic DBMS can utilize CPU resource more when there is no distributed transaction with TPC-C because the overhead of handling distributed locking and deadlocks is eliminiated. (Figure 4)
- It is often impossible for machines to get very far ahead of the slowest machine, since new transactions may have data dependencies on previous ones that access data on slow machines. (Figure 5 (a))
  - A non-deterministic DBMS can reorder transactions on demend to avoid this problem.
- The flexibility of non-deterministic DBMSs does not yield much benefit in a cluster with slow machines as we expected. (Figure 5 (a))
  - We can optimize non-determinitic DBMSs by aborting transactions (70% of local transactions).
- The performance cost of OLLP are independent of the performance cost of processing distributed transactions. (Figure 6)
- For most real-world scenario, OLLP yields very few transaction restarts. (Figure 6)
- A deterministic DBMS still scales better than a non-deterministic DBMS even on a high contention scenario because the non-deterministic DBMS needs to handle distributed deadlocks. (Figure 8)
