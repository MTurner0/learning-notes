# [Delta Lake: High-Performance ACID Table Storage over Cloud Object Stores](https://15721.courses.cs.cmu.edu/spring2023/papers/20-databricks/p975-armbrust.pdf)

## About



**Media type:** Research paper, *Proceedings of the VLDB Endowment*

**Year:** 2020

## Summary

- A **transaction** is a series of read/write operations sent to a database, and it's generally desirable for transactions to be *[*ACID**](https://15445.courses.cs.cmu.edu/spring2023/slides/15-concurrencycontrol.pdf)-compliant.

    - **Atomicity:** Either every operation in a transaction is successfully performed, or nothing is.

        - Most DBMSs log all actions so that aborted transactions can be undone.

    - **Consistency:** A *consistent* database satisfies application-specific correctness constraints, and a *consistent* transaction maps a database from one consistent state to another consistent state.

    - **Isolation:** Each transaction interacts with a database as though it were the only active transaction.

        - **Concurrency control** protocols allow DBMSs to decide the proper interleaving of operations from multiple transactions.

    - **Durability:** All changes of committed transactions should be persistent.

        - Logging or [shadow paging](https://courses.cs.washington.edu/courses/csep545/01wi/lectures/class5/sld002.htm) can be used by the DBMS to ensure that changes are durable.
    
- 