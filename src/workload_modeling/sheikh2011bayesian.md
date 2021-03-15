# ICAC'11 - A Bayesian Approach to Online Performance Modeling for Database Appliances using Gaussian Models

:::info
- Muhammad Bilal Sheikh, Umar Farooq Minhas, Omar Zia Khan, Ashraf Aboulnaga, Pascal Poupart, David J Taylor
- Institutes
    - University of Waterloo
- ICAC'11
- https://dl.acm.org/doi/10.1145/1998582.1998603
:::

### Background

- Database Appliance
    - A VM with a pre-installed copy of a OS and a DBMS
    - Easy to deploy

### Motivation

- DBA may need to predict workloads to decide how to allocate resources
- Previous work on this
    - Analytical models
        - Need a domain expert
        - Specific to a particular DBMS
    - Experiment-driven
        - Method
            - Modeling workloads by sampling from query executions
            - Use statistical models to fit the workloads
        - Problems
            - Any new change to the workloads make these previous methods need to collect new data and retrain their models.
            - Hard to introduce prior knowledge to the models.

### Problem

To model workloads with Gaussian Process and make it adapt to changing workloads fast.

#### Formal Definition

Assumptions:
- Each query belongs to a particular query type $Q_i$, where $1 \le i \le T$.
- There are $T$ types of queries.
- A mix of queries $m_j$ is represented as a vector $<N_{1j},...,N_{Tj}>$, where $N_{ij}$ represents # of queries in type $Q_i$.
- The total number of queries in a mix is less than $M$, where $M$ is defined by the DBA.
- The samples for the mix $m_j$ is represented as $S_j = <m_j,r_{ij}>$ where $r_{ij}$ is the real response time for a query in type $Q_i$ in mix $m_j$.

Goal:

To find a function $f(.)$ such that $\hat{r}_{ij} = f(m_j, Q_i)$ where $\hat{r}_{ij}$ is the estimated response time for a query in type $Q_i$.

### Main Idea

It maintains two models:
- Response Time Model
    - Given the current workload mix $m_i$ and the target query type $Q_i$, predict the response time.
- Configuration Model
    - Given the current system configuration, predict the parameters of the response time model.

With these models, the system will not need to retrain for new system configs because it can predict the parameters from the configuration model.

### Overview

![](https://i.imgur.com/8jmV23I.png)

### Each Components

#### Generating Training Data

Two ways:

- Uniformly sampling # of queries for each query type
    - This will generating a data set with a small variance and its total load would concentrate on $\frac{M}{2}$. Not good for learning.
- Uniformly sampling (the total number of queries, the number of different types of queries)

#### Modeling Response Time

Proposed Two Types of Models

- Linear Gaussian Models
    - Input: could be
        1. the current total load (# of queries) $l$
            => Linear Load Model
        2. the # of queries for each query type $m = <N_{1},...,N_{T}>$
            => Linear Query Mix Model
    - Output: the response time $r$
    - Model: $P(r|l;\theta) = \mathcal{N}(\beta_0 + \beta_1 l, \sigma^2)$
    - How to learn? Maximum Likelihood Estimation (MLE)
- Gaussian Process Models
    - Input: could be
        1. the current total load (# of queries) $l$
            => Gaussain Process Load Model (GPLM)
        2. the # of queries for each query type $m = <N_{1},...,N_{T}>$
            => Gaussian Process Mix Model (GPMM)
        3. Combination of total load $l$ and mix $m$
            => Gaussian Process Mix + Load Model (GPMLM)
    - Output: a gaussian distribution of the response time $r$
    - Model: Gaussain Process
        - Mean Functions:
            1. 0 mean
            2. linear mean function: $mean(x) = \beta_0 + \beta_1 x_1 + ... + \beta_T x_T$
        - Kernel Functions:
            1. Squared Exponential Function (SE, i.e. RBF Kernel)
                $$
k(x, x') = \sigma^2 exp(\frac{-||x - x'||^2}{2 \eta ^ 2 I})
                $$
            3. Rational Quadratic Function (RQ)
                $$
k(x, x') = \sigma^2 [1 + \frac{||x - x'||^2}{2 \alpha \eta ^ 2 I}] ^ {-\alpha}
                $$
    - How to find the hyper-parameters? Same as linear models, Maximum Likelihood Estimation (MLE).

#### Modeling Hyper-parameters of a Response Time Model

They found that
- A different configuration of the system needs a different set of hyper-parameters (i.e. a different model)
- If a configuration do not appear in the training data set, the model may not learn well.

So, we need a model to predict hyper-parameters for GP models.

- Input:
    - Mean of recent response time: $R_{MEAN}$
    - STD of recent response time: $R_{SD}$
    - Buffer Pool Size: $BP$
    - CPU Count: $CPU_{NUM}$
    - CPU Frequency (in MHz): $CPU$
    - Memory Size: $MEM$
- Output: each hyper-parameter used by GP models (one model per hyper-parameter)
- Model: should be Gaussain Process, but the paper does not say explictly
    - Mean and kernel functions are unknown.

### Experiments

#### Model Accuracy

Effect of Buffer Pool Size (Figure 3)
- It shows the linear models work poorer than GP models in all conditions, especially when the database fit partially in the buffer pool.

Effectiveness under overload (Figure 4)
- It shows the GP models able to capture the variance of response time even if the system is overloaded and the variance is large.

Overall Accuracy (Figure 5)
- It shows GPMLM (0, RQ) works well in all tests.

#### Online Adaptability

Online Costs
- Linear models work poorly so it does not adapt it to the online setting.
- GP models with linear mean have very high cost since the mean function has $T + 1$ hyper-parameters to learn.
    - Takes 1 hour to learn for 22 query types with 500 samples/type.
- GP models with 0 mean and RQ kernel works best.
    - Takes 4~7 minutes to learn for 22 query types with 500 samples/type.

Adapting to Dynamic Configurations (Figure 6)
- This experiment evaluates how the model performs when the configuration changes.
- If each time the configuration changes and the model simply throw all samples, the results show it suffer from high error rate in the beginning.
- If the model does not throw the samples but keeps them, the results show the error rate would be much lower at the same time.
- The results also show that the online models work similar to the models pre-trained using the same workload.

Adapting to Dynamic Workloads (Figure 7)
- This experiment evaluates how the model performs when new query types appear in the workload.
- The model that keeps the old data while collecting new data works best.

#### Model Robustness

Impact of New Queries (Figure 8)
- GPMLM(0, RQ) works best with 4% increase in precentage error when there are 5 new query types.
    - This is because GPMLM also models the total number of queries which is a useful info for predicting response time.

Online Model Convergence (Figure 9)
- This experiment tests how well GPCM works when a new query type appear
- Setting the hyper-parameters to 0 works worst.
- Setting the hyper-parameters by averaging over existing parameters work quite well.
    - This shows that there are correlation between these parameters
- Setting the hyper-parameters using GPCM works best.
- It also shows that 100 samples are enoguh for a good GP model.

Configuration Model Accuarcy (Figure 10)
- I don't understand this...

### Notable References

- Use Gaussian Process to model workloads
    - EDBT'11 - Predicting completion times of batch query workloads using interaction-aware models and simulation.
    - SIGMOD'10 - iTuned: a tool for configuring and visualizing database parameters.

### Conclusion

- Pros
    - GP's advantages
        - Can introduce prior knowledge (distributions)
        - Can provide confidence intervals for each prediction

### Questions

- Why not just use online-learning methods to overcome dynamic workloads?
- It seems like it assumes that every query in the same type has the same response time or at least similiar time. Is this a reasonable assumption?
- How does the second way of sampling decides the ratio of number of queries between each type when generating training data?
- What is the difference between the kernel functions that this paper uses?
- Why is training the configuration model more reasonable than training a single response time model?
    - because the response time model can only work for one type of system configuration.
    - the space of system configurations is much smaller than the space of possible workloads.
