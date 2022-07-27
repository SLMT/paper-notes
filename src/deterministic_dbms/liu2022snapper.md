# SIGMOD'22 - Hybrid Deterministic and Nondeterministic Execution of Transactions in Actor Systems

- Authors: Yijian Liu, Li Su, Vivek Shah, Yongluan Zhou, Marcos Antonio Vaz Salles
- Institute: University of Copenhagen, Denmark
- Published at SIGMOD'22
- Paper Link: <https://dl.acm.org/doi/10.1145/3514221.3526172>

## Background

Now there are many applications using actor programming models:

- Games
    - Halo 4
    - League of Legends
- Telecommunication
    - Ericsson
- E-commerce
    - Paypal
    - Walmart
- IoT

They have the demand of transactions such as:

- Purchasing equipment in games
- E-commerce

To fulfill transaction requirements for actors, Akka introduces **transactors**, which includes the ideas of:

- Two-phase Locking
- Two-phase Commit
- Early lock release

Actor-oriented Databases (AODBs)

- Why?
- Why not general-purpose DBMS?
    - AODBs are more scalable
- 


### Orleans

Orleans is a framework for actor models. Key features:

- Virtual actors
- Asynchronous message passing
    - But the order of messages is non-deterministic (may be out-of-order)
- Reentrancy
    - An actor is allowed to interleave multiple requests when some requests are waiting asynchronous operations.

## Motivation

However, the current design of transactions in actors makes all transactions become distributed transactions, even if the actors are at the same machine. This introduces significant amount of overhead to transactions.

This paper finds that some transactions of actor systems especially fit the idea of determinism, because all the parameters and participating actors are known in advance for those transactions. Determinism can greatly reduce the overhead of actor systems.

But, some other transactions still need to be executed non-deterministically, so how to make both execution work in a single system become a challenge.

## Problem

To design an architecture that can execute transactions in an actor system in both deterministic and non-deterministic modes.

A transaction is defined as a series of method invocation to multiple actors issued by an actor and requires **conflict serializability** and **durability**.

### Assumptions

Environments:

- Single machine
- Actor models

A transaction executed in the deterministic mode must provide:

- The main actor (who issue the transaction)
- The first method to be invoked and corresponding inputs
- The set of actors that this transaction is going to access

## Method

### System Architecture

- Coordinator actors
- Transactional actors 
- Loggers
    - Multiple loggers
    - Each logger has its own log file
    - Transactional actors sends its log to one of loggers decided by a simple hash function

## Experiments

### Base Settings

- Environments
    - AWS EC2
        - 4-core 3.0 GHz CPU
        - 10.5 GB Memory
        - 16GB SSD with 8K IOPS
- Benchmarks
    - TPC-C
        - Only NewOrder transactions
        - Each warehouse is an actor, and the stock table is partitioned into multiple actors
    - SmallBank
        - Add a new type: MultiTransfer transactions - transferring money from one account to multiple accounts
        - Each account is an actor

### PACT vs. ACT

#### Impact of Transaction Size

Varying transaction sizes with SmallBank's MultiTransfer transactions.

Throughput (Fig.12)

- Low transaction size -> low contention
    - PACT needs more message exchanges -> slower -> lower throughput
- High transaction size -> high contention
    - ACT aborts more transactions -> lower throughput
- Logging
    - PACT can write logs in batches due to deterministic batching -> more efficient

Latency (Fig.13)

- PACT's medium latency is almost the same as ACT's
    - Only when size = 64, PACT has higher medium latency due to the delay of batching
- ACT has higher 99th latency because of dynamic reordering of non-deterministic locking

Conclusion

- PACT has more predictable latency and higher throughput in high contention workloads
- ACT does better only in low contention workloads

#### Impact of Workload Skewness (Fig.14)

Deciding the keys/actors of transactions using Zipfian distribution with varying parameters. It also compares PACT & ACT with Orleans' Txn. Orleans' Txn is basically ACT but with early lock release and timeout deadlock avoidance.

- Orleans' Txn loses in all kind of workloads even without deadlocks (explained in the next section)
- ACT has lower throughput in higher skewness which makes sense.
- PACT has higher throughput in higher skewness because batching become more efficient.

#### Comparing ACT with Orleans' Txn (Fig.15)

Comparing the latency of ACT with Orleans' Txn using a special type of transactions, each of that may do NO-OP to actors to test the overhead of maintaining a transaction in both systems.

- OrleansTxn has higher overhead in calling an actor and 2PC.

### Hybrid Execution (Fig.16)

Running SmallBank with transaction size = 4

Throughput

- Hybrid execution yields lower throughput than the expectation. Reasons:
    - PACTs force ACTs to wait for batch processing
    - PACTs are blocked until the previous ACTs are committed
    - ACTs aborts more due to conflict with PACTs
- This situation becomes worse in high-skewed workloads

Latency

- PACTs generally have higher latencies
- As PACTs become less, PACTs run faster because of smaller batches, which has lower possibility to be blocked.
- As ACTs become less, more long-latency ACTs are aborted due to higher possibility to conflict with PACTs.

Aborts

- Most aborts come from read/write conflict of ACTs and serialiability check between PACTs and ACTs.

### Scalability (Fig.17)

SmallBanks

- PACTs have better scalability in skewed workloads
- All methods scale linearly

TPC-C

- PACTs have better scalability in skewed workloads
- All methods scale linearly
- PACTs and ACTs have much lower throughput than NT due to inefficient logging methods.

## Conclusion

## Questions

- ❓ What are 'transactors'? why do we need them?
- ❓ How does this paper make deterministic and non-deterministic execution co-exist in a single system?
- ❓ Why does Orleans use 'virtual actors'?
- ❓ Conflict Serializability
- ❓ How to ensure serializability while deterministic and non-deterministic txns are presented?
    - See the example in Figure 8
- ❓ Why did you use hybrid execution instead of non-deterministic only?
- ❓ How often do deadlocks happen in hybrid execution? Looks like it is a big problem.
- ❓ Do batch IDs of PACTs and txn IDs of ACTs come from the same counter?
- ❓ Does DBMS have the cases of non-serializable transactions not due to deadlocks?
