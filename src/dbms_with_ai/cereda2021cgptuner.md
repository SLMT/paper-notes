# CGPTuner: a Contextual Gaussian Process Bandit Approach for the Automatic Tuning of IT Configurations Under Varying Workload Conditions

- Authors: Stefano Cereda, Stefano Valladares, Paolo Cremonesi, Stefano Doni
- Institute:
    - Politecnico di Milano, Milan, Italy
    - Akamas, Milan, Italy
- Published at VLDB'21 (Vol. 14, No. 8)
- Paper Link: <http://vldb.org/pvldb/vol14/p1401-cereda.pdf>

## Background

A modern DBMS has hundreds of tunable configurations. Selecting a proper set of configurations is crucial for the performance of the system.

## Motivation

- Hundreds of parameters => large search space
- We also need to consider the parameters of IT stacks (e.g., OS, Java VM) to maximize the performance
    - which means more parameters to tune
- The same parameters may also not behave in the same way in different workloads (see Figure 1)

![Figure 1](cereda2021cgptuner-fg1.PNG)

Figure 1: Cassandra under different YCSB workloads while varying two configurations

## Problem

Goal: To design a tuning algorithm able to consider the entire IT stack and continuously adapt to the current workload.

## Previous Work

- Needs to collect data offline
    - iTune
        - Uses Gaussian Processes to approximate the performance surface with different configurations
        - Con: Learned knowledge cannot be transferred between workloads, which means that we need to rebuild the model for each workload.
    - OtterTune
        - Has ability to reuse the past experience in other workloads to a new unseen workloads
        - Con: Requires to collect large amount of data set (over 30k trials per DBMS, about several months)
- Online Learning Methods
    - OpenTuner
        - Uses multiple heuristic search algorithm and dynamically selects the best one
        - Con? Unknown (TODO)
    - BestConfig
        - Iterative sampling strategy
        - Con? Unknown (TODO)

## Method

## Experiments

## Conclusion

## Questions

- What does `vm.dirty_ratio` do?
- It seems like OpenTuner has already used multi-armed bandits to solve tuning problems. What are the differences between it and this work?
