# [*The Datacenter as a Computer*, Third Edition](https://pages.cs.wisc.edu/~shivaram/cs744-readings/dc-computer-v3.pdf)

(Ch. 1-2)

## About

Certain applications (*internet services*) are large enough to require the resources of one or more data centers to run them.

**Media type:** Book

**Year:** 2019

## Summary

- *Warehouse-scale computers* run large, complex internet services across a massive scale of software infrastructure, data repositories, and hardware platform.

    - Are a type of data center, but not the only type.

        - Traditional data centers host computing systems for multiple teams or companies within a single location.

        - Relative to the traditional data center, WSCs run a smaller number of larger applications.

    - Many internet services use multiple data centers.

        - Potentially, each data center is a complete replica of the same service.

        - There's a large gap in connective quality between intra- and inter-data center communications.

- Architecture of a WSC

    - **Servers:** the hardware building blocks of WSCs.

        - Mounted within racks and interconnected with local Ethernet switches.

    - **Storage:** Disks and Flash SSDs.

        - [DAS vs. NAS](https://en.wikipedia.org/wiki/Direct-attached_storage).

    - *[Networking fabric](https://en.wikipedia.org/wiki/Computer_network#Network_topology):* How are individual computational nodes connected to each other?

        - Both abstractly (i.e. topology) and physically (i.e. [transmission media](https://en.wikipedia.org/wiki/Data_transmission)).

    - **Buildings:** house the computing, network, and storage infrastructure. Building design can influence the performance of the WSC:

        - Power delivery.

        - Cooling.

- *Platform-level software:*

    - To facilitate the abstraction of a collection of individual servers as one WSC, all individual servers need certain common firmware, kernel, OS distribution, and libraries.

    - *OS-level virtualization:*

        - *[Containerization](https://en.wikipedia.org/wiki/Containerization_(computing)):* A program only has access to the resources allocated to a certain container.

        - *[Virtual machine](https://en.wikipedia.org/wiki/Virtual_machine):* A virtualized computer system that provides the functionality of a physical computer.   

            - **I/O virtualization:** Since a VM does not have direct access to hardware resources, I/O requests go through an abstraction layer, which incurs performance overhead.

            - **Availability model:** [Live migration](https://en.wikipedia.org/wiki/Live_migration) is used by cloud providers to move running VMs out of the way of planned maintenance events, ensuring high availability.

            - **Resource isolation:** Malicious VMs can exploit multi-tenant features to interfere with shared resources, making the tradeoff between security guarantees and resource sharing important.

- *Cluster-level infrastructure:*

    - Resource management software controls the mapping of user tasks to hardware resources, enforces priorities and quotas, and provides basic task management services.

    - Large-scale distributed applications require reliable distributed storage, [RPCs](https://en.wikipedia.org/wiki/Remote_procedure_call), message passing, and cluster-level synchronization.

    - Application framework software controls data partitioning, distribution, and fault tolerance.

- *Application-level software:*

    - System design should be general-purpose, since services often have diverse requirements and product requirements often evolve.

- *Monitoring and development software:*

    - Service-level dashboards are often used to monitor signals used to characterize service health and notify operators (or automated systems) to take corrective action.

    - Distributed system tracing tools aim to facilitate performance debugging by determining all work done in the system on behalf of a given initiator and detail relationships (causal or temporal) among the components used.

        - [Black-box monitoring systems](https://doi.org/10.1145/1135777.1135830) observe networking traffic among components and use statistical inference to guess causal relationships.

        - Instrumentation-based tracing schemes annotate modules to pass along tracing information.

- WSC software tradeoffs:

    - WSC vs. desktop:

        - Pros (i.e. advantages for software development complexity in WSCs compared to desktop systems):

            - Massive parallelism.

            - Homogeneous computing platform.

        - Cons: 

            - Scale.

            - Hardware faults.

            - Speed of workload churn.

    - Buy vs. build: How much of your software is third-party?

    - *Tail latency:* Travel time of the slowest requests.

## Highlights

Trends:

1. Data access from memory is [increasingly expensive](https://arxiv.org/pdf/2012.03112.pdf); memory bandwidth is growing slower than capacity.

1. CPU speed per core is flat.

1. SSD, NVMe are replacing HDDs.

1. Ethernet bandwidth is growing.
