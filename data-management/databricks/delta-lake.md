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
    
- Object store APIs:

    - Cloud object stores use a *key-value store interface.*

        - *Object:* A binary blob up to a few TB. Updates usually require rewriting the entire object.

        - Users create *buckets*, each of which can store multiple objects.

        - Each object is identified by a *key* string, often modeled after a filepath.

        - Metadata APIs (e.g. [`LIST`](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjects.html) the keys of all objects in a bucket) are usually slow.

    - Consistency properties:

        - Usually: [eventual consistency](https://learn.microsoft.com/en-us/azure/cosmos-db/consistency-levels) for each key guaranteed, with no consistency guaranteed across keys.

        - Time delays between when one client uploads a new object and when the object is visible to other users.

    - Approaches to tabular datasets in object stores:

        - Store records as a [collection of columnar items](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=5447738 "See HDFS"), potentially partitioned into directories based on attributes.

            - Problems: No atomicity, _eventual_ consistency, poor performance, no management functionality (e.g. table versioning or audit logs).

        - [Custom, "closed-world" storage engines](https://event.cwi.nl/lsde/papers/p215-dageville-snowflake.pdf "Such as the Snowflake data warehouse") that manage metadata in a separate, strongly-consistent service.

            - Problems: Running separate, highly-available service to manage metadata can be expensive, add overhead, and lock users into one provider.

    - Performance characteristics of object stores have implications for analytical workloads:

        1. Keep frequently-accessed data close-by sequentially.

            - Generally favors columnar formats.

        1. Since updates rewrite objects, objects shouldn't be too large.

        1. LIST operations should be avoided, with key ranges preferred where possible.

- *Delta lake table:* On a cloud object store or file system, a directory holding:

    1. *Data objects:* [Apache Parquet objects](https://parquet.apache.org/docs/overview/motivation/) containing relation tuples.

        - Advantages of Parquet: Column-oriented, offers diverse compression updates, supports nested data types, has performant implementations in many engines.

        - Each data object has a unique name, usually assigned by the author as a GUID.

    1. *Log:* A sequence of records (each taking the form of a JSON object), together with occasional checkpoints. Stored in the `_delta_log` subdirectory within the table.

        - *Checkpoint*, for specific log objects: Summarizes the log up to that point in Parquet format. 

## Commentary

The way people [talk about](https://www.confessionsofadataguy.com/5-things-i-wish-i-knew-about-databricks-before-i-started/) the Databricks ecosystem reminds me of Apple products: each individual unit does its job well but only reaches its full potential as part of the complete tool system.

