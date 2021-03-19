# Hihooi: A Database Replication Middleware for Scaling Transactional Databases Consistently

- Authors: Michael A. Georgiou, Aristodemos Paphitis, Michael Sirivianos, Herodotos Herodotou
- Institute: Cyprus University of Technology, Limassol, Cyprus
- Published at TKDE'20
- Paper Link: <https://ieeexplore.ieee.org/abstract/document/9068420>

## Motivation

Previous appraoches focus on scaling-out by data partitioning, but most of applications do not have large amount of data. It is not necssary to store data in multiple machines.

They propose to scale-out by replication in a master-slave and asychronous fasion.

## Problem

To maintain a master-slave architecture on a DBMS system with high read scalability.

Main challenge: How to replicate data efficiently and ensure strong consistency?

## Method

(Quick read through)

Statement replication:
1. Exceutes the SQL in the primary DB
2. Record the execution order of each statement
3. Replay the statements in the same ordre in backup DBs.

## Experiments

(Not check)

## Conclusion

Pro
- Middleware approaches
- Scales well for read-heavy workloads

Con
- Not scale for write-heavy workloads since every write transactions must be executed in the primary DB once.
- Only suitable for the cast that data can be stored in a single machine

Compared to deterministic DBMSs
- No need to avoid ad-hoc queries.
- No need to know read/write-set in advance.
- However, deterministic DBMSs can deal with more general OLTP workloads.

## Questions

1. How to ensure low latency?
   - By using asychronous architecture to avoid 2PC.
2. The master is still a bottleneck when using a master-slave architecture.
   - They assume most of transactions are read transactions, which can be routed to slave nodes.
3. Why do the experiments show that Hihooi can still scale on write-heavy workloads?
