# Database Meets AI: A Survey

- Authors: Xuanhe Zhou, Chengliang Chai, Guoliang Li, JI SUN
- Institute: Tsinghua Unversity, Beijing, China
- Published at TKDE'20
- Paper Link: <https://ieeexplore.ieee.org/document/9094012>

## Learning-based Database Configuration

### Knob Tuning

Problem: to find the best set of configurations for a DBMS.

#### Search-based Tuning

Finding the best configurations by branching and bound.

- [SoCC'17 - BestConfig: tapping the performance potential of systems via automatic configuration tuning](https://dl.acm.org/doi/10.1145/3127479.3128605)
  - Method
    1. Divides the search space into smaller subspaces
    2. Sampling from the subspaces and iteractively reduces the search space to find the best one
  - Cons
    - Heuristic, no guarantee to find the best one
    - The search space is too large

#### Traditional ML-based Tuning

Finding the bets configurations using traditional ML-based methods.

- [SIGMOD'17 - Automatic Database Management System Tuning ThroughLarge-scale Machine Learning](https://www.cs.cmu.edu/~dvanaken/papers/ottertune-sigmod17.pdf)
  - Alias: OutterTune
  - [Read Note](./aken2017ottertune.md)
- [EDBT'19 - SparkTune: tuning Spark SQL through query cost modeling](https://openproceedings.org/2019/conf/edbt/EDBT19_paper_226.pdf)
- Cons
  - The optimal solution obtained in the current stage is not guaranteed to be optimal in other stages.
  - Requires a large number of high quality samples for training.
  - Cannot support too many knobs.

#### Reinforcement Learning for Tunning

Uses a Reinforcement Learning (RL) agent to find the best configurations for a DBMS.

- [SIGMOD'19 - An End-to-End Automatic Cloud Database Tuning System Using Deep Reinforcement Learning](https://dl.acm.org/doi/10.1145/3299869.3300085)
  - Alias: CDBTune
  - Method
    - The RL Modeling:
      - Environment: a cloud DBMS
      - State: the internal metrics of the DBMS (similar to OutterTune)
      - Action: the values for increasing or decreasing configurations (knobs)
      - Reward: the difference of DBMS's performance
      - Agent Model: Deep Deterministic Policy Gradient (DDPG)
  - Pros
    - Does not need high-quality training data
  - Cons
    - without considering workload features
- [VLDB'19 - QTune: A Query-Aware Database Tuning System with DeepReinforcement Learning](https://www.vldb.org/pvldb/vol12/p2118-li.pdf)
  - Alias: QTune
  - Method
    - Basically same with CDBTune but considers workloads.
    - Uses Double-state Deep Reinforcement Learning (DS-DRL)

(Reading...)
