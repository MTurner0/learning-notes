# [Lakehouse: A New Generation of Open Platforms that Unify Data Warehousing and Advanced Analytics](https://15721.courses.cs.cmu.edu/spring2023/papers/02-modern/armbrust-cidr21.pdf)

## About

The *what* and *why* of [Databricks](https://databricks.com), essentially.

## Summary

- History of data architectures:

    - **Data warehouse:** A repository of integrated data used for reporting.

        - *Schema-on-write* architecture.

        - Often the container into which data are loaded in an [ETL](https://doi.org/10.1016/j.datak.2017.08.004) pipeline.

        - No ability to store and query unstructured datasets (e.g. video, audio, text documents).

    - **Data lake:** A repository of "raw" data (often files or object blobs).

        - *Schema-on-read* architecture.

        - Often the container into which data are loaded in an [ELT](https://www.astera.com/type/blog/elt-extract-load-and-transform/) pipeline.

        - Lack management features (e.g. ACID transactions) and access methods (e.g. indexes) to match warehouse performance.

    - **Cloud data lake + warehouse:** A two-tier architecture; dominant data architecture in ~2020.

        - Data are first ELed into lakes, then ETLed into downstream warehouses.

        - Separates storage and compute.
    
- Limitations of current two-tiered data architecture:

    - *Reliability:* The Lake $\to_{\text{ETL}}$ Warehouse pipeline
    
        1. is an expensive process to run every time the lake is updated.

        2. risks introducing bugs due to, e.g., differences in data lake and data warehouse engines, supported data types, SQL dialects...

    - *Staleness:* Because of the above, warehouse data is often stale compared to lake data.

    - *Limited support for complex analyses:* ML frameworks have poor interoperability with warehouses.

    - *Cost of ownership:* Continuous Lake $\to_{\text{ETL}}$ Warehouse, data stored in two places, proprietary data formats that may be difficult to migrate...

- **Data lakehouse**: low-cost and directly accessible (open-format) storage with traditional analytical DBMS features.

    - *Metadata layer:*
    
        - Raises abstraction level of data lake storage.

        - Lake itself includes a table describing which objects are parts of which tables, e.g. as a transaction log in Parquet format.

        - Allows for ACID transactions, zero-copy cloning, and table version control.

        - Can store data quality enforcement or data governance features (e.g. access control, audit logging). 

    - *Caching:*
    
        - Files from cloud object store can be cached on processing node SSDs and RAM for faster access.

        - Running transactions using the metadata layer can determine whether cached files are still valid to read. 

    - *Auxiliary data structures:*

        - Proposals for indexing "raw" lake data.

        - Column min-max statistics stored in metadata may enable data skipping.

    - *Data layout optimizations:*

        - Ordering records by individual dimensions.

        - Ordering records across multiple dimensions, using e.g. Z-order and Hilbert curves.

    - *Declarative DataFrame APIs:*

        - ML and statistics libraries use imperative code that cannot be run as SQL.

        - Using a DataFrame abstraction (compatible with, e.g., the DataFrames used in `R` and Python's `Pandas` library), this API can be made declarative by lazily evaluating transformations (which mostly map to relational algebra) and pushing the operator plan to an optimizer.
