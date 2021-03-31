# Towards a Hands-Free Query Optimizer through Deep Learning

- Authors: Ryan Marcus, Olga Papaemmanouil
- Institute: Brandeis University
- Published at CIDR'19
- Paper Link: <http://cidrdb.org/cidr2019/papers/p96-marcus-cidr19.pdf>

## Background

Query optimization is a popular and important research topic.

## Motivation

There are chances for deep reinforcement learning to help query optimization:

- Many optimization approaches are heuristics due to the complexity of the problem.
- Deep RL can learn from mistakes.

## Problem

To study if it is possible to use deep RL to generate a plan tree for a query.

## Case Study: ReJOIN

It models query planning as a deep RL problem. Each time planning for a query is an episode.

- State: relations (tables)
  - Not sure how exactly it is
- Action: which two relations to join
- Reward: the estimated cost from the query cost estimator
  - Only gives the reward when the agent reaches the final state.

## Challenges

- Large Search Space Size
  - The search space is extremely large if we want to let the RL agent deal with all the operators
- Hard to provide reward
  - To efficiently train an agent, rewards need to be dense. However, if we choose query latency to be rewards, rewards would be sparse.
  - The estimated cost is also not a good indicator for rewards because the cost estimator needs to be tuned by humans.
- High evaluation overhead
  - It is hard for the agent to come out a good plan in the beginning. It may take much longer time to evaluate the plans.

## Comments

- Is it possible to solve the evaluation problem with curriculum learning? Like starting from a easy problem.
