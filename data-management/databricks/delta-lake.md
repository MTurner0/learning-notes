# [Delta Lake: High-Performance ACID Table Storage over Cloud Object Stores](https://15721.courses.cs.cmu.edu/spring2023/papers/20-databricks/p975-armbrust.pdf)

## About

A [deeper dive](https://github.com/MTurner0/learning-notes/blob/main/data-management/databricks/lakehouse.md) into the first data lakehouse, which aims to balance provide ACID transactions, streaming I/O, and SSD caching to object stores.

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

        - Each record object contains an array of actions that produce the next version of the DLT when applied to the previous version. Actions include:

            - Change metadata: [`Metadata`](https://books.japila.pl/delta-lake-internals/Metadata/).

            - Add or remove files: [`AddFile`](https://books.japila.pl/delta-lake-internals/AddFile/), [`RemoveFile`](https://books.japila.pl/delta-lake-internals/RemoveFile/).

            - Protocol evolution: [`Protocol`](https://books.japila.pl/delta-lake-internals/Protocol/),

            - Add [provenance](https://github.com/MTurner0/learning-notes/tree/main/data-management/provenance) information: [`CommitInfo`](https://books.japila.pl/delta-lake-internals/CommitInfo/).

            - Update application transaction IDs: [`SetTransaction`](https://books.japila.pl/delta-lake-internals/SetTransaction/).
        
        - *Checkpoint*, for specific log objects: Summarizes the log up to that point in Parquet format.

- Access protocols:

    - Protocol for read-only transactions:

        1. Let the *start key* be: If [the `_last_checkpoint` object](https://books.japila.pl/delta-lake-internals/checkpoints/Checkpoints/#implementations) exists in the table's log directory, its **checkpoint ID.**
        Otherwise, **0.**

        1. Let the *reconstruction records* be a list of `.json` and `.parquet` files in the log directory found via a LIST operation with the *start key.*

        1. Use the *reconstruction records* and last checkpoint (if it exists) to reconstruct the state of the table with its data statistics.
        This can be parallelized.

        1. Use the [statistics](https://books.japila.pl/delta-lake-internals/data-skipping/#learn-more) to identify the set of data object files relevant to the query.

        1. Apply the query to the set.

    - Protocol for write transactions:

        1. Use the first two steps of the read protocol to identify a recent log record ID $r$.

        1. Use the third step of the read protocol to read the table at version $r$.

        1. Write any new data objects into new files in the correct data directories, using GUIDs to generate object names.
        This can be parallelized.

        1. Attempt to write the transaction's log into the $r+1$ `.json` log object.
        This will fail if another client has written this object.

        1. Optionally, write a new `.parquet` checkpoint for log record $r+1$ and update `_last_checkpoint` to point to this checkpoint.

    - Notes on these access protocols:

        - Step 4 of the write protocol must be atomic.

        - All transactions that perform writes are [serializable](https://en.wikipedia.org/wiki/Serializability), while the read protocol described here achieves [snapshot isolation](https://en.wikipedia.org/wiki/Snapshot_isolation).
        It is also possible to perform serializable read transactions.

        - Delta Lake currently only supports transactions within one table.

        - The rate of write transactions is limited by the latency of [put-if-absent](https://en.wikipedia.org/wiki/Optimistic_concurrency_control) operations to write new log records.

- Higher level features:

    - Time travel and rollbacks using the DLT's log.

        - Supports SQL `AS OF timestamp` and `VERSION AS OF commit_id` syntax for reading past snapshots.

        > `MERGE INTO mytable target USING mytable TIMESTAMP AS OF <old_date> source ON source.userId = target.userId WHEN MATCHED THEN UPDATE SET *`

    - Efficient `UPSERT`, `DELETE`, and `MERGE` operations compared to data lakes, since these operations can be performed transactionally.

    - Each DLT's log can be treated as a message queue, reducing the need for separate message buses (e.g. Apache Kafka, Kinesis). 

    - Data layout optimization features:

        - The bin-packing [`OPTIMIZE`](https://docs.delta.io/latest/optimizations-oss.html#optimize-performance-with-file-management) function rebalances data objects with the aim to make each one ~1GB.

        - [Z-ordering](https://docs.delta.io/latest/optimizations-oss.html#z-ordering-multi-dimensional-clustering) by multiple attributes.

        - Optional `AUTO OPTIMIZE` on [Databricks cloud](https://docs.databricks.com/delta/tune-file-size.html#auto-compact).
    
    - [Caching](https://docs.databricks.com/optimizations/disk-cache.html).

    - Audit logging using [`DESCRIBE HISTORY`](https://docs.delta.io/latest/delta-utility.html#-delta-history) and commit information logging.

    - Schema evolution and enforcement.

    - Connectors to query and ETL engines.

## Commentary

The way people [talk about](https://www.confessionsofadataguy.com/5-things-i-wish-i-knew-about-databricks-before-i-started/) the Databricks ecosystem reminds me of Apple products: each individual unit does its job well but only reaches its full potential as part of the complete tool system.
