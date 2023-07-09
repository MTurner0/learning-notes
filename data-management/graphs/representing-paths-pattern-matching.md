# [Representing Paths in Graph Database Pattern Matching](https://www.vldb.org/pvldb/vol16/p1790-martens.pdf)

## About

*Short description*

**Media type:** Research paper, *Proceedings of the VLDB Endowment*

**Year:** 2023

## Summary

- Returning paths along nodes and edges (rather than only the nodes and edges themselves) is a desireable function of graph database query languages.

    - A pair of nodes in a graph may have an arbitrarily large number of paths between them.

    - Representing intermediate path query results can be expensive when the number of paths is very large.

- A *regular path query* (RPQ) comprises a regular expression $e$.

    - The evaluation of $e$ over an edge-labelled graph $G$ returns all node pairs $(x, y)$ s.t. there exists a path from $x$ to $y$ in $G$ whose sequence of edge labels forms a word in the language of $G$.

    - For QLs in which paths are first-class citizens, the evaluation of an RPQ returns all such node pairs $(x, y)$ as well as all paths between each pair: e.g. $(x, y, \text{path}(x, t_1,\dots, t_k, y))$.

- Current common approaches to represent results of RPQs:

    - Return a relational table where each tuple represents a triple: $(x, y, p)$.

        - Issues: The table may be prohibitively large -- or even infinitely large, if a cycle connects $x$ and $y$.

    - Path restrictions:

        - **TRAIL**: No repeated edges.

        - **SIMPLE**: No repeated nodes.

        - **SHORTEST**: [Take a wild guess](https://en.wikipedia.org/wiki/Does_exactly_what_it_says_on_the_tin).

    - Neo4J presents query results in the form of a [*graph projection*](https://neo4j.com/docs/graph-data-science/current/management-ops/projections/graph-project-cypher-projection/),
    
        - i.e. A subgraph of the original graph containing only nodes and edges in the results of the RPQ.

        - Issues:

            - Can represent the nodes and edges in the results of the RPQ, but lose path-specific information.

            - Are computed from the tabular output from the RPQ, so don't actually save on compute time over tabular query output.

- The data model as an edge-labeled directed multigraph.

    - A *graph database* is a tuple $G=(N, E, \eta, \lambda)$, where:

        1. $N\subseteq \text{NID}$ is a finite set of node identifiers, and $E\subseteq\text{EID}$ is a finite set of edge identifiers;

        1. $\eta: E\to (N\times N)$ is a [total function](https://en.wikipedia.org/wiki/Partial_function) called the *incidence mapping*, which maps each edge to the nodes it connects.

        1. $\lambda: E\to\mathbf{L}$ is a total function called the *labeling function*, which maps a label to each edge.

        1. (Notation) Let $N_G = \text{nodes}(G)$, $E_G = \text{edges}(G)$, $\eta_G = \{\eta(e)| e\in E_G\}$, $\lambda_G = \{\lambda(e)| e\in E_G\}$.

        1. An *unlabeled graph* is a triple $(N, E, \eta)$ with no labeling function $\lambda$.

    - A *path* is a sequence $\rho = v_0e_1v_1e_2v_2\dots e_nv_n$, $n\ge 0,e_i\in E,\eta(e_i)=(v_{i-1},v_i)\forall i\in [n]$.

        - $\lambda(\rho)$ is the sequence of edge labels occurring on the edges of $\rho$.

        - $\text{src}(\rho)$ is the node $v_0$ at which $\rho$ starts, and $\text{tgt}(\rho)$ is the node $v_n$ at which it ends.

            - If $S,T\subseteq N$ and $\text{src}(\rho)=v_0\in S, \text{tgt}(\rho)=v_n\in T$, then we say $\rho$ is a path from $S$ to $T$.

    - A *path multiset representation* (PMR) over a graph $G$ is a tuple $R=(N, E, \eta,\gamma, S, T)$, where:

        1. $(N, E, \eta)$ is an unlabeled graph.

        1. $\gamma: (N\cup E)\to (N_G\cup E_G)$ is a total homomorphism.

            - If $R$ is a PMR of $G$ and $v\in N$, then we say that the node $v$ represents the node $\gamma(v)$ in $G$.

            - Each path $\rho$ from $S$ to $T$ in $R$ represents the path $\gamma(\rho)$ in $G$.

                - $\gamma(\rho)\coloneqq \gamma(v_0)\gamma(e_1)\gamma(v_1)\dots\gamma(e_n)\gamma(v_n)$

        1. $S,T\subseteq N$ are sets of source and target nodes, respectively.

            - A PMR $R$ is *trim* if every $v\in N_R$ is on some path from $S$ to $T$.

- Related work:

    - Factorized database representations reduce data redundancy by using compact factorized representations at the physical layer.

    - Finite state automata and [extended conjunctive regular path queries](https://homepages.inf.ed.ac.uk/libkin/papers/pods10b.pdf).

    - Query-preserving graph compression.

        - Aim is to compress the input graph $G$ w.r.t. a class of queries $\mathcal Q$, as opposed to compactly representing the output $Q(G)$ of a single RPQ $Q$.
