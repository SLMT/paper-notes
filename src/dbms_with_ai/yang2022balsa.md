# Balsa: Learning a Query Optimizer Without Expert Demonstrations (WIP)

- Authors: Zongheng Yang, Wei-Lin Chiang, Sifei Luan, Gautam Mittal, Michael Luo, Ion Stoica
- Institute: UC Berkeley
- Published at SIGMOD'22
- Paper Link: <https://arxiv.org/abs/2201.01441>

## Background

Query optimizer is an important component that finds out a good execution plan for a query.

## Motivation

### DBMS without Good Optimizer

雖然像是 PostgreSQL 這些 DBMS 都有很成熟的 Query Optimizer，但是比較新興的系統像是近年的 NewSQL 系統就沒有成熟的 Query Optimizer。 然而為每一個系統設計一個成熟的 Query Optimizer 非常花費時間。

因此如果能用一個 ML Model 來替代的話會方便很多。

### RL without Demonstration

之前已經有過許多用 RL 實作 Query Optimizer 的研究，然而這些做法都是假設有一個承受的 Query Optimizer 存在，並提供過去的經驗做為學習對象。不過在缺乏這種 optimizer 的系統上，就無法直接引用這些做法解決問題。

因此需要一種不需要有專家 (成熟的 optimizer) 也能夠用的 RL 作法。

## Problem

在沒有其他 expert optimizer 的情況下使用 RL 替代 query optimizer。

### Challenges

- 沒有 expert optimizer，這點可能會造成毫無方向的 exploration，以及大量的 exploration 時間
    - 如何避免找到很糟糕的 plan 導致大量的時間浪費
    - 如何有效率的學習避免花費大量時間尋找好的 plan

### Assumptions

- Database 資料不會改變
- 假定 query plan 會被拆成 select-project-join block 進行處理

### RL Modeling

## Previous Work

- DQ: 學習 query optimizer 的 cost model，因此 performance 被 cost model 給限制住
- Neo: 先從 query optimizer 產生過的 plan 學習，然後在實際環境中執行

這些方法都假定有 expert cost model 或者 optimizer。

## Method

這篇 paper 提出的作法就是 simulation-to-reality learning，也就是先在 simulated 的環境中學習，然後在轉移到實際環境中學習。這樣可以在 simulation 階段就避免花時間執行很差的 query plan。

## Experiments

## Conclusion

## Questions

- 真的現有的 RL 做法都需要 expert demonstration 嗎？
- 從做法看起來，其實就是先從一個簡單的 cost estimator learn 一個 estimator model，然後再利用實際環境 fine tune 這個 model。而尋找 plan 的方法仍是使用現有的 algorithm 搭配 learn 好的 estimator model 學習。這樣真的有比較好？  
