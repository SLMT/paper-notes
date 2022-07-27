# Tastes Great! Less Filling! High Performance and Accurate Training Data Collection for Self-Driving Database Management Systems

- Authors: Matthew Butrovich, Wan Shen Lim, Lin Ma, John Rollinson, William Zhang, Yu Xia, Andrew Pavlo
- Institute: Carnegie Mellon University, Army Cyber Institute, Massachusetts Institute of Technology
- Published at SIGMOD'22
- Paper Link: <https://dl.acm.org/doi/10.1145/3514221.3517845>

## Background

A self-driving DBMS usually contains a behavior modeling module which predicts the cost of a database action on a given workload.

The module needs a set of training data to train, so the system needs a method to collect these data.

## Motivation

Current training data collection scheme:

- Offline
    - Method 1: Cloning a database and simulate an existing workload trace
        - Con:
            - Cloning a database is time-consuming
            - Recording workload trace is also not easy
    - Method 2: Running a new database with synthetic queries
        - Con:
            - Needs extra time for simulating workloads (maybe days or weeks) to generate enough data for robustness
            - Cannot capture the real metrics of online environments
- Online
    - Con: overhead too high

Needs an online method with low overhead

## Problem

### Requirements

- Needs a method that collects **internal** features (CPU time, # of concurrent workers, info of GC etc.)
    - External features are not accurate
- Needs a method that collects metrics in **kernel-space**.
    - Collecting metrics in user-space is expensive due to the overhead of system calls and I/O

## Method

- How to collect metrics in kernel-space with low overhead?
    - By writing a kernel module using Berkley Packet Filter (BPF) library, which allows a user to write a kernel module without knowing much kernel knowledge.
        - Pro:
            - Has OS-level privilege
            - No need to run DBMS using root privilege
            - Faster

## Experiments

## Conclusion

## Questions
