# [Provenance in Databases: Why, How and Where](https://homepages.inf.ed.ac.uk/jcheney/publications/provdbsurvey.pdf)

## About

“Citation needed,” so to speak.
Provenance can help end users assess the validity of query results.

## Summary

> The most common forms of database provenance describe relationships between data in the source and in the output, for example, by explaining *where* output data came from in the input, showing inputs that explain *why* an output record was produced or describing in detail *how* an output record was produced.

- The trio model:

    - Why-provenance:

        - **Lineage** (of a tuple $t$ in the query output): A set of tuples present in the input that “contributed to” $t$ or “helped produce” $t$.

            - **Witness**: A subset of input DB records sufficient to ensure that a given output tuple appears in the result of a given query.

            - Witnesses are not necessarily unique.

        - **Why-provenance** (of an output tuple $t$ in the output of query $Q$ applied to a DB instance $I$): The witness basis of $t$ according to $Q$; all combinations of source tuples that witness the existence of $t\in Q(I)$.

            - **Witness basis:** A compact representation of the set of all witnesses (dependent on $Q$ and $D$).

                - Equivalent queries may have different witness bases.

                - **Minimal witness basis:** A subset of the witness basis, invariable under equivalent queries.

                    - **Minimal witness:** A witness with no proper subinstances that are also witnesses in the witness basis.

    - How-provenance:

        - **Provenance semiring:** A polynomial showing the structure of the proof that a given set $w$ is a witness of $t$.

            - Let $w=\{\{t_1\},\{t_1,t_3\}\}$ be the why-provenance of $t$. Then its how-provenance might be $h=t_1^2+t_1\times t_3$.

        - The why-provenance of an output tuple can be derived from its how-provenance, but the converse is not always true.

        - **Route:** A form of how-provenance over a schema mapping.

    - Where-provenance:

        - **Location** (of a value $v\in t:t\in I$): The tuple (row) and attribute (column) address of $v$ within the relation $I$.

        - **Where-provenance** (of some $v\in t: t\in Q(I)$): The set of locations of $v$ in $I$.

        - The where-provenance of $v$ consists of locations that can be found in tuples of the why-provenance of $t$.

- Computing provenance:

    - **Eager computation:** Rewrite the query so that extra, provenance-relevant annotations are carried to the output database during transformation.

        - Requires performance and space overhead for all query execution.

        - Useful when source data is unavailable after transformation.

    - **Lazy computation:** Compute provenance post-transformation, on request.

        - Deriving provenance post-transformation from the source database, output database, and transformation requires sophisticated reasoning.

        - Does not require changing query execution, so useful when that is not possible.
        
        - Useful when output storage space is limited.

## Highlights

_Below is a review of this paper that I wrote for [CS 784](https://pages.cs.wisc.edu/~paris/cs784-s22/)._


### What are some applications of provenance?

In general, provenance information describes where data originated and how it has been manipulated.
This information contextualizes query outputs.

In data warehouses, provenance information can be used to check the correctness of transformation steps in the ETL pipeline.
Provenance information can be useful for debugging big data pipelines, where the size of the data, the complexity of analytics pipelines, and long runtimes can make step-wise debugging time- and resource-expensive.
Additionally, provenance information can be used to determine the minimum set of tuples in the source database whose deletion will remove a corresponding output tuple from a view. 

### What is the difference between why, how, and where provenance?

First, consider the notion of a *witness*.
A witness of some output tuple $t$ (with respect to a given query $Q$ and database instance $I$) is a subset of the input database records that is sufficient to ensure $t$ appears in the query output.

The *why-provenance* of $t$ is the witness basis of $t$ according to $Q$ -- that is to say, a set of input tuples that prove the existence of $t$ in $Q(I)$.

The *how-provenance* of $t$ is a \textit{provenance semiring}: a polynomial expression that describes the structure of the proof that a certain set of input tuples forms the witness basis of $t$ according to $Q$ (i.e. the why-provenance).
How-provenance is therefore closely linked to why-provenance.
Indeed, the why-provenance of $t$ can always be derived from the how-provenance of $t$, although the converse is not always possible.

The *where-provenance* of a value $v: v\in t, t\in Q(I)$ is the set of tuple-attribute (i.e. row-column) locations describing where $v$ can be found in $I$.
If $v\in t$, then the where-provenance of $v$ consists of locations that can be found in the tuples of the why-provenance of $t$.

### Discuss the differences between eager and lazy provenance computation.

*Eager provenance computation* approaches rewrite queries so that extra, provenance-relevant annotations are carried to the output database during transformation.
The rewriting and carrying require performance and storage overheads, respectively, to compensate for the extra information.
However, eager approaches are useful when source data is unavailable post-transformation.

*Lazy provenance computation* approaches derive provenance information post-transformation, on-demand.
This requires no query rewriting nor additional output storage space, so lazy approaches are useful when query execution cannot be modified or when output space is limited.
However, deriving provenance post-transformation from the source database, output database, and transformation requires sophisticated reasoning, and is not possible if source data is unavailable post-transformation.