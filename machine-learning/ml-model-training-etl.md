# [ML Model Training Is An ETL](https://managingml.substack.com/p/ml-model-training-is-an-etl)

## About

## Summary

- Elements of ETL pipelines:

    1. Source of data;

    2. Transformation;

    3. Output artifact that a downstream use case may depend on.

- Examples:

    - Unsorted data is ingested to an indexed DB table.

        - A user submits a query to the table and receives an answer.

    - Sensitive customer data is transformed to preserve data privacy, returning a hashed and scrubbed table.

        - An unauthorized user submits a data request to the table and receives a safe, compliant result.

    - Examples of purchase fraud are used to train a model to be a fraud prediction tool.

        - An account hijacker attempts to make a fraudulent purchase attempt and is denied.

- Why describe ML model training in terms of ETL?

    - Leverage ETL DevOps tools for model observability and diagnostics.

    - Rely on ETL infrastructure (versioning, compliance resources, shared responsibilities and enforced obligations on upstream dependency steps of a pipeline) for model training.

    - Directors and executives may understand ETL better than ML, so framing model training as a special case of ETL may improve communication.