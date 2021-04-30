# RLIRank: Learning to Rank with Reinforcement Learning for Dynamic Search

- Authors: Jianghong Zhou, Eugene Agichtein
- Institute: Emory University, Atlanta, USA
- Published at WWW'20
- Paper Link: <https://dl.acm.org/doi/10.1145/3366423.3380047>

## Background

Dynamic search is an iterative process to rank documents and collect feedbacks from a user in order to come out the best ranking that fits the query provided by the user.

## Motivation

They claim that the previous work that uses learning to rank (LTR) methods fail to capture all the ranked documents' information to improve the overall quality of ranking.

## Problem

To design a RL agent that ranks documents iteratively for dynamic search.

## Method

### RL Modeling

State:
- A sequence of (\\(d\\), \\(q\\)) pairs. 
  - \\(d\\): the embedded vector of a ranked document
  - \\(q\\): the embedded vector of the current query

Action:
- \\(a_r\\): a picked document
  - This action updates the state by adding the picked document to the sequence
- \\(a_t\\): the action to update the query by the feedback from the user
  - This action updates the state by replacing all the queries with the new query

Reward: NDCG or \\(\alpha\\)-NDCG

RL Method:
- Choosing the action with the max expected reward (not total reward).
- The expected reward 

### Document and Query Embedding

Uses Google Universal Sentence Encoder

## Experiments

Looks great. However, it shows this method outperforms MDP method. Why?

## Conclusion

This is definitely not a usual RL approach. I am not sure why it works.

## Questions

- Why did it use stacked RNN?
- What is the value network presented in the paper? Is that a DQN method?
  - It seems like "the value network" is a network to predict the reward given the current state and action. So, it is not a DQN method.
- It uses NDCG to calculate the rewards. What is that?
- Why is its loss function to minimize the relevance scores? Why not just maximizing rewards?
- What is MDP in the experiments? Does it mean Markov Decision Process?
- \\(a_t\\) depends on user's feedbacks. How does the RL agent iterate each action before it receives user's feedbacks?
