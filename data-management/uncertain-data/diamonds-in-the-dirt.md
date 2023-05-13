# [Probabilistic Databases: Diamonds in the Dirt](https://cs.stanford.edu/people/chrismre/papers/cacm-paper-full.pdf)

## About

A review paper about probabilistic databases.
The titular saying “diamonds in the dirt” here refers to extracting useful information from noisy data.

## Summary

- Extractors produce **confidence scores** to represent uncertainty: e.g. The probability scores in [BERN2](http://bern2.korea.ac.kr/).

- **Probabilistic inference:** [The probability that a propositional expression is true](https://core.ac.uk/download/pdf/82296807.pdf).

- Formalisms:

 - **Block independent-disjoint (BID) database:** Tuples within a relation can be partitioned into **blocks** such that the tuples *within* each block represent disjoint events while tuples *across* blocks are independent.

 - **c-table:** Each tuple is annotated with its **lineage** (a Boolean expression over some hidden variables).

  - *Closure property* of c-tables: it is always possible to compute the lineage of output tuples from those of the input tuples.

- How do we represent a probabilistic DB?

 - Tradeoff between representational accuracy and processing/scaling.

 - A **probabilistic database** is a discrete probability space $PDB=(W,\mathbf{P})$, where $W=\{I_1,I_2,\dots,I_n\}$ is a set of possible instances, called possible worlds, and $\mathbf{P}:W\to[0,1]$ is such that $\sum_{j=1,n}\mathbf{P}(I_j)=1$.

 - If $Q$ is a query and $t$ is a possible tuple in the answer to $Q$, then $\mathbf{P}(t\in Q)$ denotes the probability that $t$ is an answer to $Q$ in a randomly chosen world.

 - For $Q$, the job of a ProbDMS is to return all possible tuples $t_1,t_2,\dots$ along with their probabilities $\mathbf{P}(t_1\in Q),\mathbf{P}(t_2\in Q),\dots$

- How do we answer queries?

 - Complex, [decision-support style](https://en.wikipedia.org/wiki/Decision_support_system) SQL with aggregates.

 - Query execution requires:

  1. Fetching and transforming the data.

  2. Performing probabilistic inference. (This part dominates total running time as data scales.)

 - A **safe plan** allows probabilistic inference to be performed entirely within the query plan, with no need for a separate probabilistic inference step.

  - Relational algebra operators must be extended to manipulate probabilities.

  - Safety does not depend on the database instance; a safe plan can be executed on any instance.

  - Safe queries have PTIME complexity, while most (see the *dichotomy property*) unsafe queries are #P-hard.

 - **Materialized views** are answers to previous queries that can be used to answer future queries, potentially improving performance.

- How do we present query results to the user?

 - Output tuples can be ranked by probability, but this does not represent the probabilistic relationship between two output tuples (correlated, mutually exclusive, etc.).

 - User-specified ranking criteria may be combined with probabilities $\mathbf{P}(t_i\in Q)$ to rank output tuples.

  - **Top-k query answering:** Returning only the k highest-ranked tuples.

 - In SQL, aggregates come in two forms:

  - **Value aggregates:** For ProbDMS, this is an expected value.

  - **Predicate aggregates:** Can be computed by provenance semi rings combined with safe plans.


## Highlights

_Below is a review of this paper that I wrote for [CS 784](https://pages.cs.wisc.edu/~paris/cs784-s22/)._

### What are some key applications of probabilistic databases?

Probabilistic databases are useful for noisy and uncertain datasets.
Since there is no shortage of noisy and uncertain data in the world, there is no shortage of applications for probabilistic databases.

Applications for probabilistic databases can be summarized by the source of uncertainty in the data.
When data represent measurements from instruments (such as sensor and RFID data), the measurement error introduces uncertainty.
For data representing model output (such as natural-language processing methods used for information extraction), model limitations introduce uncertainty.
Data inaccuracy, incompleteness, and inconsistency introduce uncertainty, necessitating data cleaning.
Probabilistic databases are well-suited to representing uncertain data, particularly if that uncertainty can be quantified.

### What are the challenges to scaling probabilistic databases?

There exists a trade-off between representational fidelity and scalability.
Associations between the events represented by the tuples within a probabilistic database may be complicated in reality, but faithful representation of those associations may not be conducive to query evaluation.

Furthermore, executing queries in probabilistic databases requires fetching the data, transforming it, and performing probabilistic inference.
As the size of the data scales, the probabilistic inference dominates the total running time.
Performance can be improved by integrating probabilistic inference with the other steps, especially for safe queries.

### Could probabilistic databases be useful for machine learning applications? Why or why not?

Probabilistic databases can be used to store results from machine learning models where the results have some describable amount of uncertainty.
As mentioned above (and discussed in detail in the paper), information extraction systems quantify the uncertainty in their results with a probabilistic confidence score, which can be used by probabilistic databases.
Indeed, many machine learning models (some of which are used for information extraction tasks such as named-entity recognition) annotate their predictions with confidence scores, which means their output could be stored in a probabilistic database.