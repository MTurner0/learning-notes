# [MapReduce: Simplified Data Processing on Large Clusters](http://static.googleusercontent.com/media/research.google.com/es/us/archive/mapreduce-osdi04.pdf)

## About

MapReduce is a programming model facilitating fault-tolerant, parallelized, distributed computations.

**Media type:** Conference paper, [OSDI](https://dl.acm.org/conference/osdi)

**Year:** 2004

## Summary

- Motivation:

    - Conceptually simple computations on large input data benefit from distribution of tasks across many machines.

    - Desire to abstract away the details of parallelization, fault-tolerance, data distribution, and load-balancing.

    - MapReduce is such an abstraction.

- The *Map* function transforms key/value pairs to a set of intermediate key/value pairs.

    - Input keys and values are from a different domain than intermediate keys and values.

    - `(k1, v1) -> list(k2, v2)`

    - Input data are partitioned into $M$ splits, which can be processed in parallel by different machines.

        - $M$ is typically chosen such that each split is between 16MB and 64MB.

        - A worker assigned a map task retrieves key/value pairs from its input split, applies the user-defined *Map* function to each pair, and buffers intermediate key/value pairs in memory.

- The *Reduce* function merges together the set of intermediate values corresponding to each intermediate key, usually producing just zero or one output value per key.

    - Intermediate keys and values are from the same domain as the output keys and values.

    - `(k2, list(v2)) -> list(v2)`

    - Buffered intermediate key/value pairs are periodically written to local disk and partitioned to $R$ regions.

        - $R$ is typically chosen to be a small multiple of the number of worker machines expected to be used.

        - Local locations of buffered pairs are transmitted to the master.

        - A worker assigned a reduce task is informed by the master of buffered data locations and uses remote procedure calls to read the data from the map workers' local disks.

        - Once the reduce worker has read all its assigned intermediate data, it sorts by intermediate keys then passes each key and corresponding intermediate values to the user-defined *Reduce* function.

        - Each reduce worker appends output from each *Reduce* iteration to its own global output file.

- Distributed execution:

    - For each map task and reduce task, the master stores the state (*idle*, *in-progress*, or *completed*) and the identity of the worker machine (for non-idle tasks).

    - The master labels a worker as *failed* if it has received no response from the worker for a certain length of time.

        - Completed *and* in-progress map tasks on a failed worker are reset to *idle* state and rescheduled.

            - Since the output of map tasks is stored on workers' local disks, this output is inaccessible on failed workers.

        - Only in-progress reduce tasks are reset to *idle* on a failed worker, since output from completed tasks is stored globally.

        - Map and reduce task outputs are committed atomically.

            - When the *Map* and *Reduce* operators are deterministic with respect to their input values, **the distributed implementation** (even in the presence of faults) **produces the same output as would have been produced by non-faulting sequential execution of the entire program.**

            - If the *Map* and/or *Reduce* operators are non-deterministic, **the output of any particular reduce task** $R_i$ **is equivalent to the output for** $R_i$ **produced by a sequential execution of the non-deterministic program.**

- [Assumptions](https://pages.cs.wisc.edu/~shivaram/cs744-fa22-slides/cs744-mapred.pdf):

    - Commodity networking, less bisection bandwidth.

    - Failures are common.

    - Local storage is cheap.

    - Replicated filesystem.

    - Input is splittable.