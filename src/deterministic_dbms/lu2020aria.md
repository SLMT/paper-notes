# Aria: A Fast and Practical Deterministic OLTP Database

- Authors: Yi Lu, Xiangyao Yu, Lei Cao, Samuel Madden
- Institute: MIT
- Published at VLDB'20
- Paper Link: <http://vldb.org/pvldb/vol13/p2047-lu.pdf>
  - Video: <https://www.youtube.com/watch?v=DvgMjPPB134>

## Background

Deterministic DBMS show great potential for optimizations in transaction processing.

## Motivation

Currently, deterministic DBMSs all request the input transaction requests to provide their read-sets and write-sets. If not, they will need to execute the transactions once to determine their read-/write-sets.

## Problem

To design a concurrency control mechanism without knowing read-sets and write-sets while ensuring deterministic execution.

## Method

- Main Idea: Batch execution with barriers
  - Execution phase:
    - Executes one batch of transactions at a time.
    - Every transaction runs in parallel, reads from the same snapshot, and writes to its local buffer.
    - Updates to indices are also buffered, so there is no phantom due to index updates.
  - Commit phase:
    - To commit a transaction, it must wait until all other transactions finish execution as well. (barrier)
    - If there is a WW, RW, or WR conflict with earlier transaction, aborts and reschedules the later transaction.
- Optimization: Deterministic Reordering
  - Uses a relaxed check while deciding aborts:
    - Aborts a transaction only if:
      - It has WW conflict with an earlier transaction.
      - Or, it has at least one RW conflict and also at least one WR conflict with earlier transactions at the same time.
        - This rule prevents cycles in the dependency graph. (proved in Section 5.3)
- Optimization: Fallback Phase
  - If too many transactions are aborted, add a fallback phase after the commit phase.
  - The fallback phase will execute the aborted transactions in the Calvin fashion.
    - The key is that we have known the read-sets and write-sets of the aborted transactions because the system has executed them once.

## Experiments

### My Expectation

- Aira works well in low contention workloads but poorly in high contention workloads.
  - It works actually ok in high contention workloads since it has fallback strategies.

### Experiment Summary

- 8.2 YCSB
  - Aira works great since the keys of YCSB transactions are drawn from a uniform distribution.
- 8.3 Scheduling Overhead
  - Aira has almost no scheduling overhead since the only overhead is to book-keeping writes in a reservation table.
- 8.4 Effectiveness of Deterministic Reordering
  - The performance of Aira goes down as the workload becomes more skew, however, Aira still performs better than Calvin thanks to fallback phases.
  - Aira with deterministic reordering also shows its effectiveness compared to Aira without DR.
- 8.5 TPC-C
  - Interestingly, this experiment shows how contention affects Aira significantly in a standard OLTP benchmarks.
- 8.6 Distributed Transactions
  - Aira basically outperforms all baselines no matter how many distributed transactions are there.
  - However, note that the contention in the TPC-C setting is extremely low, which gives Aira a big advantage.
- 8.7 Scalability
  - Aira scales well.

## Conclusion

Pros

- It won't need read-sets and write-sets for deterministic execution.
- It performs much better than Calvin in low contention workloads.

Cons

- Aborts many transactions in high contention workloads.
- Barriers between batches leads to slow down the entire transaction execution when transaction lengths are imbalanced.

## Questions

- Is that possible to use wound-wait or wait-die 2PL to achieve the same effect?
  - No, this may lead to nondeterministic execution since there is no barrier.
- The system aborts all the transactions that conflict with the earlier transactions in the same batch. So, does this mean that we better run this system in a low contention workloads?
  - Yes. See experiments in Section 8.5.
- How about long transactions that do not have conflicts with others? Does Aria suit the workloads with these transactions?
  - Figure 5 verifies this concern. If there are a few long transactions in a batch, it will greatly slow down the system.
- Why do they need barriers?
  - Consider the case that T1 does not run at all and T3 starts to commit in Example 1 of the paper. T3 may not find out T1 does not run since the system does not have T1's write-set. This makes the database state nondeterministic.
  - It also makes all transactions can run in parallel during the commit phase since all information that need to be checked are set during the execution phase.
