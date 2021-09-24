# Scalable Multi-Query Execution using Reinforcement Learning

- Authors: Panagiotis Sioulas, Anastasia Ailamaki
- Institute: EPFL
- Published at SIGMOD'21
- Paper Link: <https://dl.acm.org/doi/10.1145/3448016.3452799>

## Background

### Vectorized Execution

Vectorized execution 是一種藉由 SIMD 來加速 query execution 的作法。 SIMD 的特色在於可以藉由一道 instruction 同時對多個資料進行相同操作，可以大幅增加平行性。 Vectorized execution 則是為了要使用 SIMD 進行 query execution，必須設計特別的 algorithm 把要處理的資料轉成 vector，然後對這些 vector 進行 SIMD 操作來完成 query execution。

常見可以做 vectorized execution 的動作包括：

- Scan with filtering
- Hash Table Probing
- Histogram Building

Reference: Andy Pavlo 的 [Vectorized Execution 課程](https://15721.courses.cs.cmu.edu/spring2020/schedule.html)。

### Work-Sharing

- Global Query Plan: a shared plan for multiple queries
- Online sharing: 線上一邊接受新 query，一邊將 query plan 與執行中的 query 合併來減少資源花費。
    - 關鍵問題在於：已經在執行的 query plan 是無法更改的。因此新進來的 query plan 只能配合執行中的 query plan 偵測相似的 sub-plan。然而實際上考慮所有 query 的可能 query plan 的時候，是有可能找到更好的 global query plan，但 online 作法的限制錯失了這個機會。
    - [SIGMOD'05 - QPipe](https://www.pdl.cmu.edu/PDL-FTP/Database/qpipe.pdf)
        - 早期做 work-sharing 的方式
        - 簡單地偵測並 reuse 之前的 query result 或 intermediate result
        - 通常都是看是否有拿過相同 range 的資料等等
    - [SIGMOD'10 - DataPath](https://faculty.ucmerced.edu/frusu/Papers/Contribution/2010-sigmod-datapath.pdf)
        - 嘗試將執行中的 query 與剛進來的 query 的 plan tree 合併，變成一棵 global plan tree，然後中間就有些部分可以 reuse。可以視為是將 common 的 sub-plan 組合起來。
    - [VLDB'09 - CJOIN](https://dslab.epfl.ch/pubs/cjoin.pdf)
        - 考慮將 operator reordering，確切來說會考慮優先將 selectivity 低的放前面，然而最佳來說並非是最好的做法。
    - [SIGMOD'02 - CACQ](https://dsf.berkeley.edu/papers/sigmod02-cacq.pdf)
- Offline sharing: 藉由在給定的 query batch 中嘗試所有可能的選項，以找到 cost 最低的選項。
    - 這些做法的問題都在於問題的 solution space 太廣，導致只要 query 一多 complexity 就會變高。以致於 scalability 不佳。
    - Multi-query Optimization (MQO)
        - 很多 work 都在解這個問題。
        - 基本作法就是盡可能地遍歷所有 query 的可能 operator 組合，以找出最佳的 query plan。
        - 每種做法的差異在於 bounding case 不同
    - [VLDB'14 - Shared-workload Optimizers (SWO)](http://www.vldb.org/pvldb/vol7/p429-giannikis.pdf)
        - 跟 MQO 的差異在於並非是以 batch of queries 做 input，而是還考慮了在一個 workload 中，每一種 query 出現的頻率。
- 應用的系統目前都是基於 SWO，所以會有 scalability issue
    - SharedDB
    - MQJoin

### Adaptive Query Processing

利用 query execution 中搜集到的資訊適度地動態調整 query plan。

- [Symmetric Hash-join](https://www.youtube.com/watch?v=jveohy_qhHU)
    - 一般的 hash join 是先對其中一個 join table 建立 (join key -> record id) 的 hash table，然後再一一拉出另一邊 join table 的 record 來在 hash table 中尋找 match。
    - Symmetric Hash-join 則是對兩邊 join table 都建立一個 hash table，通常應用於 streamming query engine。因為不確定哪一邊的 table 資料會先過來，所以最好兩邊都建 hash table，然後讓一邊資料來的時候去查另一邊的 hash table。
    - 缺點是需要花費大量記憶體建 hash table，因此一般的 DBMS 不會使用這種做法，通常只用於 stream process。
- [SIGMOD'00 - Eddies](https://dsf.berkeley.edu/cs286/papers/eddies-sigmod2000.pdf): 藉由觀察 operator 的 input 與 output 來動態 reorder operator
    - Eddies 的概念是將 query plan 裡面先後順序可以替換的 operator 打散 (例如 hash join，任何的 join order 可能都不影響結果)，然後由 eddies 的 routing algorithm 來決定今天進來的一個 tuple 應該優先做哪一種 join。
    - Eddies 的 algorithm 會隨著狀況判斷每一個 tuple 該先進哪一個 operator (例如先做哪一個 join)。判斷的方式為記錄 operator 的 input 與 output 數量，如此一來可以知道先做哪一個 operator 可能比較有利。
- [ICDE'03 - State Modules (STeMs)](https://dsf.berkeley.edu/papers/icde03-stems.pdf)
    - 如果我理解沒錯的話，就是一個 hash table
    - 主要應用是在多重 SHJ，原本的 3-way 以上的 SHJ 在越上層的 join 就需要建越大的 hash table，因為越上層的 intermediate result 越大，而且 SHJ 要求 join 兩側都要建 hash table。然而 state modules 搭配 eddies 使用的話，就不需要建儲存 intermediate results 的 table。只需要為每一個 base table 建 hash table 就好。Eddies 會控制如何 join 這些 bash table。

### Reinforcement Learning

這篇使用 Q-Learning 應用在 reorder operator。

### Learned Cardinality Estimation

這篇利用之前 Learned Cardinality Estimation 相關的研究成果來預測 cardinality。

## Motivation

## Problem

### Assumtions

- OLAP Workloads
    - Almost no update to the database
    - Queries intend to summary the statistics of the database
- 只針對 select-project-join (SPJ) 的情況優化，其他則維持原本的處理方式
    - 進一步假設這些 SPJ 的 query plan 都出現在整個 query plan 的最底層

## Method

### Main Contribution

- 設計出 RL-based 的 tuple router (eddy)，來強化 online work sharing 的效果，以找到更好的 global query plan。

### 概念

- Episodes
    - 每個 episode 取得一個 table 的 vector (vector size = 1024 tuples)，eddy 建立一個 global query plan，然後轉交給一個 executor 的 worker thread 做處理。

### Architecture

- Main DBMS
    - 負責接收使用者 query 並轉成初步的 plan tree
    - 得到 plan tree 之後會將 SPJ 的 sub-plan 送進 RouLette 處理，而這個 sub-plan 會用另一個 RouLette 的 place holder 替代。
    - DBMS 等待 RouLette 將 SPJ 的 tuples 送回，送回之後繼續執行 SPJ 以外的 query plan
- RouLette
    - Ingestion Module
        - 負責從 DBMS 的 storage engine 索取 table 的資料，目標是用來 scan table
        - 索取時以 vector 的形式取出，以使用 vectorized execution 的技巧優化
        - 自己一個 thread
        - 會為所有 ongoing 或者 incoming 的 query 所需的 table 的資料
        - 會追蹤每一個 query scan 每一個 table 的起點，如此就可以知道針對某一個 query 來說是否已經 scan 完所有資料
        - 每次輸出的 vector 上的每一個 tuple 會包含一個 bit set，紀錄該 tuple 要輸出給哪一些 query，如此一來後面的 component 就知道結果該輸出給哪些 query
        - Scan 的時候使用 round-robin 的方式公平地掃每一個需要的 table，以盡可能地服務到所有 query。
    - STeMs
        - 負責使用 in-memory index 暫存每一個 table 輸出的資料，以讓 Eddy Module 可以以任意順序存取需要的資料。而不是依照原本 plan tree 的邏輯存取。
    - Eddy
        - 負責在每一個 episode 產生一個 global query plan 處理所有 ongoing query 需要的資料。
        - 會在 episode 之間動態調整 policy 以在之後的 episode 產生更好的 global plan
        - 實作 selection push-down strategy。因此產生的 global plan 一定是 selection 在最底層，然後才是 join 與 projection。
        - Join 的 plan 會使用 multi-step optimization (MSO) 來產生，其使用的 policy 則是由 RL 在 episode 之間學習。
        - 持續記錄每一個 operator 與 query 的 pair，operator 的 input 與 output，作為 state 供 RL 學習。
    - Executor
        - 有一個 worker thread pool，每一個 worker 負責處理一個 episode，一個 episode 包含輸入一個 table 的 vector 並執行 global query plan。
        - 執行流程
            1. 收到 Ingestion 傳來的 input vector
            2. 進行 selection
            3. 插進對應的 STeM (hash table)
            4. 執行 Symetric Join
            5. 將結果的 tuple 回傳到 Main DBMS 給對應的 query plan
        - 執行時採用 vectorized execution

### Core Problems

- How does Eddy generates a global query plan?
- How does a worker thread efficently execute the query plan?

### Eddy's Global Plan Geneartion Algorithm

1. 接收一個 input vector
2. 找出與該 input vector 的 base relation 可以 join 的 relation，作為 candidates
3. 選出最佳的 candidate relation 做 join
4. 紀錄已經 join 的 relation 與符合這次 join 的 query set
5. 加入新 join 的 table 的 selection (可能會有多種 selection 同時存在，因為要針對不同的 query 的 constraint 做處理)
6. 繼續尋找 candidate 與 join
7. 直到完成某一個 query join 的要件，紀錄該條 path 最後的 output 要傳遞至哪些 query
8. 往回尋找分歧點 (導致某些 query 不符合的 join 點)，繼續尋找其他 candidate 並 join
9. 直到所有 query 都有符合的 path 後結束

選擇 candidate 時需要考慮的問題：

- 盡可能讓越多 query share 越多 join 越好
- Join Selectivity
    - 任兩個 relation join 之後會有多少 record 是 match 的
- Join 後的資料量

### Candidate Selection Policy

為了盡可能讓 Eddy 選擇最好的 candidate，它必須要使用以下幾項技術：

- Cost Estimation
    - 概念是預測每一個 Operator 的 cost，這個 cost function 的 input 是 operator 的 input 與 output size
    - 將 plan 的所有 operator 的 cost 全部加起來就是 plan 的 cost
    - Operator Cost 這篇定義為 computation cost，並且假定 linear to input size。Cost function 為：$K_a * n_{in} + \lambda_a * p(o) * n_{in}$，其中 $K_a、\lambda_a$ 為常數，$p(o)$ 是 operator 的 selectivity。
        - 這篇論文對所有相同類型的 opeartor 使用相同的 $K_a、\lambda_a$ (join, selection)，數值是從過去的統計中計算出來的
- Policy Goal: 找到一個 plan 的 total cost 是最低的
    - 這個 goal 可以帶換成 RL 想要找到 total reward 是最高的
    - 這件事情不容易做到的原因在於，我們是一步步把 candidate 接起來，所以在接前面的 operator 時，並不知道後面的 operator 會有哪些，而且會造成多少 cost。
    - 另一種做法是可以 iterate 所有的 possible plan，但這樣在 online 做就太花時間。
- RL Modeling
    - State:
        - Virtual vector
            - 代表這個 step 已經 join 的 relation 與符合條件的 query
            - 如果有多條 path (對應不同的 query)，則 stack 起來變成長 vector
        - Input size
    - Action: 針對最上面那條 path 的 candidate 裡面選一個 operator
    - Reward: 選擇這個 candidate operator 所帶來的 cost (包含 join cost 與 selection cost)

### Opeartor Implementation

- Selection
    - 每一個 tuple 都會有一個 query-set bitmap，代表這個 tuple 會用於那些 query。
    - 每一個 selection operator 會對每一個 tuple 計算另一個 bitmap，然後把 tuple 本身的 bitmap 與這個 bitmap 取 AND。得到新的 bitmap。
    - 在 predicate evaluation 的時候，會事先將所有 query 的 predicate 分成多個 range，其中每一個 range 會對應到一組 bitmap，並使用 binary search 的方式找 match 的 range，其 bitmap 就是該 tuple 的結果。
- Join
    - 使用 SeTMs 來 join，join 完後再對兩者的 bitmap 取 AND 來決定要留下來些 record。
- Join Pruning

## Conclusion

Interesting problem and idea, but the in-memory assumption is not realistic.

## Questions

- Who are using work-sharing? Any practical examples?
    - 就 paper 的理論來看好像大多還是用在 stream processing，但 batch processing 的情況也能夠使用。
- 如果 where 條件不同也能夠 work sharing 嗎？有例子嗎？
    - 可以，這篇論文的 Figure 8 就是在說明如何處理 where 不同的情況。
- Section 3 一開始提到 "Ingestion pulls a vector from the host’s storage into RouLette."，為什麼是拉出 vectors？
    - 為了做 vectorized execution
- 如果我理解沒錯的話，RouLette 是否只有針對 Select-Project-Join 的 case 優化？
    - 是，這篇論文只探討 Select-Project-Join 的優話
- 如果我理解 STeMs 沒錯的話，就是一個有 index 的 in-memory data table。這是不是代表需要耗費大量記憶體暫存資料？但是 Data Warehouse 的資料通常很大，這些資料要如何暫存，記憶體空間肯定是不夠吧？
    - 第三章最後有提到 STeMs 的實作採用 in-memory 的方式。因此記憶體大小會影響 RouLette 能處理的資料量。
    - 另外也提到他們以 column store 的方式實作，所以拉取資料時只拉取有興趣的 column，以減少需要儲存的資料量。
- 甚麼時候會移除 STeMs 的資料？
    - 看起來整個 RouLette 的處理方式還是以 batch processing 為主，所以當這個 batch 處理完之後，就會刪除所有的 intermediate records (STeMs 的資料)。
- 每個 episode 都要重建 global query plan，但又只用來 process 一個 vector of tuples，這真的會快嗎？
    - 可能是因為它使用了 RL 的方式建立 query plan，所以基本上都是 O(1) 的 time complexity。另外 vector 大小也會影響重建 query plan 的次數。Paper 寫說他們 vector size 使用 1024，所以 episode 的數量並非到非常誇張的地步。
- 為什麼可以將 query processing 切成 episode？這樣修改 query plan 不會出錯嗎？
  - 不會，這邊是用到 symmetric hash join 的做法
- RL 的 cost estimation 為什麼重要？

## Slides Logics

- Background
    - Query Processing
        - 簡單複習一下流程：parsing query -> optimize query plan -> query execution
    - Multi-query Processing in OLAP workloads
        - 多個 query 同時處裡的時候，有機會可以共用一些資源
        - 舉例：兩個 query 可能有 overlap 的 record set
    - 問題：然而當產生的 query plan 差距太大時，可能就難以利用到這種機會
    - 這篇論文目標
        - Input: batch of queries
        - Goal: find a way to execute these queries fast
        - Assumption: main memory is large enough to fit the working set for the queries
- Previous Work
    - Online work-sharing methods
        - QPipe: heuristics to reuse query result or intermediate results
        - DataPath: detect common sub-plans between multiple queries
        - CJoin: consider reordering some operators to find common sub-plans
            - TODO: need an example to show why it may not be optimal
    - Off-line work-sharing methods
        - MQO: iterate all possible query plan to find the optimal global plan for a set of queries.
        - SWO: similiar to MQO, but considers the frequency of query types.
- RouLette
    - Key Idea
        - Global select-join query plan: 為所有 query 組成一個 global query plan，其中只考慮 select 跟 join 的優化。
        - Episodes: 將 query processing 切成多個 episodes，每個 episodes 處理一組 tuples，並產生獨立的 global plan。
            - 這利用到了 Adaptive Query Processing (SIGMOD’00 - Eddies) 的概念，該論文提出應該一邊執行 query 一邊修正 query plan。但該論文只考慮 single-query。
            - 好處：這樣可以逐步找到最好的 global query plan。
        - Uses RL to improve global query plan
    - System Overview (use examples to illustrate)
        - Workflow Graph
        - Main DBMS
            - Parse queries
            - Generate a query plan
            - Take out the select-join sub-plans to the RouLette engine
            - Wait for the RouLette engine outputs results for further processing
        - RouLette Engine
            - Ingestion Module: scan each table and keep tracking the progress of scan for each queries
                - Output a vector of tuples for a table (vector size: 1024)
            - Eddy Module: generate a global query plan
                - Idea: Selection -> Join (selection-push-down)
                - Steps:
                    - Put the selection operator for the tuples first 
                        - Filter based on all where constraints
                        - Uses a query-set bitmap
                    - Put insertion operator to a temp table (STeM)
                    - Select a candidate join operator (based on RL)
                    - Final Plan Tree
            - Executor Module: executes the query plan using a worker thread
                - Can run multiple episodes with multiple worker threads
    - RL Agent to select candidate operators
        - List state, action, reward
