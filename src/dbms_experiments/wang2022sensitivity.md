# VLDB'22 - A Study of Database Performance Sensitivity to Experiment Settings

- Authors: Yang Wang, Miao Yu, Yujie Hui, Fang Zhou, Yuyang Huang, Rui Zhu, Xueyuan Ren, Tianxi Li, Xiaoyi Lu
- Institute: The Ohio State University
- Published at VLBD'22
- Paper Link: <https://dl.acm.org/doi/abs/10.14778/3523210.3523221>

## Background

Many DBMS articles compare their systems with other baselines using the TPC-C benchmarks or YCSB, but these benchmarks have many tunable parameters.

## Motivation & Problem

This paper tries to find out "how sensitive are their evaluation results to these parameters and will their conclusions hold under a different setting?"

## Method

### Reproduced Systems

They use the source code from the original paper to reproduce the system. A system is considered as reproduced as long as:

1. The reproduced numbers are reasonably close to those in the corresponding article or have an explainable deviation
2. the conclusion of the article still holds

Reproduced systems list:

- Transactional DBMSs
    - Focusing on multi-core single-machine
        - Silo: an in-memory OCC system that eliminates the need of global TIDs by using a serializable commit protocol.
        - Cicada: an in-memory OCC & MVCC system that solves a global critical section and proposes varies optimizations for MVCC.
    - Focusing on distributed transactions
        - Using RDMA
            - DrTM: a DBMS that optimizes distributed transactions using RDMA.
            - GAM: a DBMS that further improves the efficiency of processing distributed transactions using RDMA.
        - Without using RDMA
            - Focusing on geo-distributed databases
                - TAPIR: a DBMS that optimizes geo-distributed transactions by merging 2PC and Paxos.
                - Janus: a DBMS that does similar things like TAPIR but also reduces abort rates using a global dependency graph.
            - Not especially focusing on geo-distributed env.
                - Calvin: a classical deterministic DBMS.
                - Star: a DBMS that optimizes distributed transactions using replicas.
                - Aria: a deterministic DBMS without the need of knowing read-/write-sets in advance.
- Key-value DBMSs
    - HERD: a KV store using RDMA
    - MICA: a KV store using DPDK (a better TCP)

## Experiments

### The TPC-C Benchmarks

They tried to answer the following questions:

- Considering commercial systems are tested with **wait time** and research prototypes are not, how much difference does it make?
- Comparing to commercial systems, most research prototypes are tested with a small number of warehouses. What is the impact of **the number of warehouses**?
- Different prototypes are evaluated under different degrees of concurrency. Will their conclusions still hold under a different **degree of concurrency**?
- What is the difference between running transactions as **stored procedures** and running them as interactive SQL transactions?
- Considering some research prototypes only run two types of transactions (i.e. New-Order and Payment), what is **the impact of the remaining three types**?

#### Impact of Wait Time

Key Findings

- Due to the restriction of wait time (21 seconds in average) and number of clients (10) to each warehouse, the upper bound of expected throughput is 0.48 txns/sec per warehouse.
- With wait time
    - Almost no contention
    - Bottleneck becomes I/O
- Most systems focus on improving performance under contention, so it is not suitable for them to use wait time.

#### Impact of Contention Level

Key factors for contention

- Number of warehouses (with a fixed number of clients)
- Number of clients (with a fixed number of warehouses)
- Percentage of cross-warehouse transactions
    - Cross-warehouse transactions have higher chance to contend with more transactions

Key findings

- Whether adjusting contention level by changing the number of warehouses work depends on if contention is a bottleneck (check by if the CPU is underutilized)
    - If contention is a bottleneck, it does work.
    - If contention is not a bottleneck (e.g., Silo, DrTM, and GAM), increasing the number of warehouses causes the throughput decreases.
- Many systems are sensitive to the number of warehouses. This may change the conclusions of the papers.
    - E.g., Aria outperforms Calvin in low contention environment, but not in high contention env.

#### Impact of Network and Disk I/O

Key solutions to avoid I/O impact throughput:

- Faster I/O: using fast hardware and mechanisms, e.g., RDMA
- Smart scheduling: e.g., early lock release
- Separate I/O: e.g., Calvin decouples replicating transactions from transaction execution

Experiments (Figure 3) show that I/O impacts scalability.

#### Impact of Not using Stored Procedures

Challenges of using interactive SQL statements:

- Hard to know read-/write-set in advance
- Adding more network round-trips
- Adding overhead of parsing SQLs

Key findings (Figure 4)

- Using stored procedures is much faster
- However, increasing concurrency level (e.g., increasing number of warehouses) can hide the above cost since it is easier to pipeline transactions.

Finding read-/write-set from interactive SQL statements is an interesting research direction.

#### Impact fo Transaction Types

Key findings (Figure 5)

- Using only New-Order and Payment can singnificantly increase throughput to most of systems
- However, it is not the case for Janus because
    - the bottleneck of Janus is not CPU
    - New-Order and Payment may have cross-warehouse behavior which increases the overhead

#### Summary of TPC-C

Tuning guidelines

- Test disk throughput => ensure data does not fit into DRAM
- Test network stack => run interactive transactions, 2PC, Paxos with low contention (many warehouses, but fit into DRAM)
- Test concurrency control => high contention (low number of warehouses)

### The Yahoo! Cloud Serving Benchmarks (YCSB)

They tried to answer the following questions:

- Since some systems have network stacks and some do not, how does network stack affect the result?
- How does the skewness of keys in the workload affect the result?
- How do the number and size of KV pairs affect the result?
- How does the read/write ratio affect the result?

#### Impact of Network Stack

Key findings:

- (Table 3) Network stack efficiency: RDMA > DPDK > TCP/UDP
- (Figure 6d) Large KVs with RDMA may be slow for reading records since reading requires using RDMA SEND, which is slower than RDMA WRITE.

#### Impact of Skewness

Key findings:

- (Figure 6a, 6b) High skewness makes the systems having concurrency control slower.
    - (Figure 6d, 6e) But not for the systems that do not need currency control (H-Store-like design), it only causes load imbalances. (Note that there is no transaction in YCSB)

#### Impact of Number of KVs

Key findings:

- (Figure 6a, 6b) the impact of number of KVs is not significant.
    - (Figure 7) This is due to the design of Zipfian distribution. Even if raising the number of KVs from 1M to 100M, the frequency of accessing the hottest key only decreases from 6.5% to 4.8%.

#### Impact of Read/Write Ratio

Key findings:

- (Figure 6a, 6c) more writes => slower
    - since reads can be processed concurrently
- Some systems that need to replicate writes (e.g., Star) gets higher impact with higher write ratio.

#### Summary of YCSB

Tuning guidelines

- Test network bandwidth => using large KVs with low contention
- Test network stack => using small KVs with low contention
- Test KV lookup speed => using small KVs with low contention but batching requests
- Test concurrency control => high contention

## Suggestions

- Running experiments under a variety of settings is better.
- An article should provide an explicit explanation about the implication of the experiment settings they use.
- TPC-E may be a better choice over TPC-C but most of transactions require non-primary indexes lookup.

## Questions

- How does Janus decouple I/O from critical sections?