# STAR: Scaling Transactions through Asymmetric Replication

- Authors: Yi Lu, Xiangyao Yu, Samuel Madden
- Institute: MIT CSAIL
- Published at VLDB'19
- Paper Link: <http://www.vldb.org/pvldb/vol12/p1316-lu.pdf>

## Motivation

Cross-partitions transactions hurt scalability of distributed database systems due to two-phase commit.

## Problem

To design a better execution scheme to avoid executing cross-partition transactions in a distributed way.

### Assumptions

- A partitioned distributed DBMS
- One of the nodes has enough memory capacity for a complete replica.

## Method

1. Separate the transactions into two categories:
   - Single-partition transactions
   - Cross-partitions transactions
2. Separate machines in the cluster into two categories:
   - Partial-replica machines
   - Full-replica machines
3. Then, divide the execution into two phases:
   - Partitioned Phase
     - Executes only single-partition transactions.
     - Each partition has a partial-replica machine as its primary machine.
     - A thread takes the responsibility to execute single-partition transactions on a partition.
   - Single-master Phase
     - Executes only cross-partition transactions.
     - A full-replica machine will be the master node for all the transactions.

## Conclusion

- Pros
  - Eliminates distributed transactions
- Cons
  - This method assumes that there is a machine which has high computing power to execute transactions and high memory capacity to store all the data in memory.
  - During phase transition, it requests all participants to synchronize with each others. This may be unrealistic for cross-WAN settings.
    - On the other hand, Calvin only needs a part of machines to reach a consensus and replicates inputs.

## Questions

- Q: How about deterministic DBMSs?
  - The paper argues that the total ordering for deterministic DBMSs is costly.
  - But, is the replication fence of STAR not costly?
