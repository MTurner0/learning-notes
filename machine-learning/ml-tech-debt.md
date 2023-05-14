# [Hidden Technical Debt in Machine Learning Systems
](https://proceedings.neurips.cc/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf)

## About

AI/ML models are all dependent on data processing infrastructure.

**Media type:** Conference paper, NeurIPS

**Year:** 2015

## Summary

- Complex models erode abstraction boundaries in ML systems, since desired behavior is dependent on external data.

    - The **CACE principle:** Changing Anything Changes Everything

        - Components of ML systems are entangled, i.e. intrinsically dependent on each other.

        - This includes input signals, hyper-parameters, learning settings, sampling methods, and convergence thresholds.
    
        - Therefore, it's difficult to isolate improvements.

    - *Correction cascades.*
    
        - If model $m_a$ exists for problem $A$ and problem $A'$ is similar to $A$, then one may quickly learn a model $m'_a$ that takes $m_a$ as input and learns a small correction to address problem $A'$.

        - However, $m'_a$ is now dependent on $m_a$.

    - *Undeclared consumers.*

        - Predictions from a model may be visible to and used by consumers without the knowledge of ML system engineers.

        - Changes to the model may therefore impact other systems in ways that may be unintentional, poorly understood, and detrimental.

- Garbage in, garbage out.

    - System may be dependent on unstable data.

    - Some input signals may be underutilized, providing little incremental modeling benefit.

        - **Legacy features** have been made redundant by new features over time but have not been removed from the model.

        - **Bundled features** are all used as inputs, even though some may be useful and others not.

        - **$\epsilon$-features** are included because they make the error metric go down or the performance metric go up, even when the improvement is very small or when including them increases overhead.

        - **Correlated features** $X$ and $Y$ may both be included (or only $Y$ included) in a model predicting $Z$, even if $X$ directly causes $Z$ while $Y$ is merely strongly correlated with $X$, which may be problematic if the correlation between $X$ and $Y$ changes.

- Live ML systems influence their own behavior over time.

    - *Direct feedback loops:* models influence selection of their future training data.

    - *Hidden feedback loops:* two ML systems influence each other indirectly through the world.

- ML system anti-patterns:

    - **Glue code:** Supporting code to extract data from or store data into general purpose packages.

    - **Pipeline jungles:** Incremental addition of information sources to the ML system resulting in bespoke pre-processing pipelines for each data source: "a jungle of scrapes, joins, and sampling steps, often with intermediate files output."

    - **Dead experimental codepaths:** Keeping code for experimental models present and executable despite lack of use. (See the role of dead model [Peg Power](https://www.sec.gov/litigation/admin/2013/34-70694.pdf) in the SEC charge against Knight Capital.)  

    - *Abstraction debt,* especially regarding distributed learning: No widespread agreement on the right interface to describe a stream of data, a model, or a prediction.

- Common ML system smells:

    - **Plain-Old-Data Type Smell:** Losing context by encoding information used in and produced by ML systems with plain data types.

    - **Multiple-Language Smell:** Cobbling together code from different languages to take advantage of different libraries and syntaxes at the potential cost of making testing more difficult.

    - **Prototype Smell:** Regular reliance on prototype environments.

- Principles of good configuration systems:

    - It should be easy to specify a configuration as a small change from a previous configuration.
    
    - It should be hard to make manual errors, omissions, or oversights.
    
    - It should be easy to see, visually, the difference in configuration between two models.
    
    - It should be easy to automatically assert and verify basic facts about the configuration: number of features used, transitive closure of data dependencies, etc.
    
    - It should be possible to detect unused or redundant settings.
    
    - Configurations should undergo a full code review and be checked into a repository.

- Systems need to respond to change in the external world.

    - Decision thresholds need to be updated.

        - This can be automated by having models learn thresholds from evaluation on heldout validation data.

    - Live monitoring of system behavior is crucial for long-term system reliability.

        - Monitoring whether the distribution of predicted labels matches the distribution of observed labels can help diagnose prediction bias.

        - If systems take real-world actions, broad action limits should be set and enforced. (This would have prevented the case of Knight Capital.)

        - Processes upstream and downstream of the ML system should be monitored, with any alerts from one node being broadcasted downstream.

- Other potential sources of debt:

    - Not testing input data.

    - Failure of systems to reproduce results.

    - Large numbers of models running concomitantly.

    - Segregation of ML research and engineering.

- Questions to assess technical debt:

    1. How easily can an entirely new algorithmic approach be tested at full scale?

    1. What is the transitive closure of all data dependencies?

    1. How precisely can the impact of a new change to the system be measured?

    1. Does improving one model or signal degrade others?

    1. How quickly can new members of the team be brought up to speed?