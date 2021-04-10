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
- [EDBT'19 - SparkTune: tuning Spark SQL through query cost modeling](https://openproceedings.org/2019/conf/edbt/EDBT19_paper_226.pdf)

(Reading...)
