# DBMS Query Processing

- SIGMOD'20 - Thrifty Query Execution via Incrementability
  - <https://dl.acm.org/doi/abs/10.1145/3318464.3389756>
  - Problem: to study how to efficiently evaluate a query even before all the data are ready.
    - Then, the query can be executed faster when all data are set.
  - Motivation: previous work only focus on select-project-join-aggregate queries, but not more complex queries such as nested queries and outer/anti-joins.
  - Assumption: data arrival rate can be predicted from historical statistics
