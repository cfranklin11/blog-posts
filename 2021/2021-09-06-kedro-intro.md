I. Kedro: Python data-science framework
  - Key features
    - CLI tool for setup, running pipelines, building
    - Separation of concerns: data sets, pipelines, nodes
    - Tooling for bundling pipeline into package for distribution
    - Plugin ecosystem for customisation & integrations (e.g kedro-docker, kedro-airflow)
II. Reasons for using Kedro: Very subjective & based on limited knowledge of the alternatives
  - Full, batteries-included framework
    - Jupyter/IPython integration
    - Fixed file structure
    - Interact with pipelines via CLI or in the Python code
    - Most of the others are libraries focused on writing & running data pipelines
    - Disadvantage: learning curve
  - Pipelines are DAGs composed of functional nodes
    - Directed Acyclic Graph
    - Encourages small transformation functions with single-responsibility
    - Key advantage of FP in general: less dependency on state
    - Older pipeline libraries (e.g. Airflow, Luigi) depend on I/O
    - Dagster seems to have similar advantage
  - Data sets integrate seamlessly into pipelines
    - Same syntax as functional nodes
    - I/O is abstracted away
IV. What a Kedro project looks like
  - File structure: separation of concerns, then domain concepts
  - Data sets
    - Defined in YAML config files
    - Handle read/write automatically based on type (e.g. S3 object, CSV file, image file)
  - Nodes
    - Functions that take data input, do something with it, then return some data
    - Do not perform I/O (that's what data sets are for)
  - Pipelines
    - Composed of nodes and other pipelines
    - Uses string labels to define inputs and outputs of nodes and connect them together into the DAG
      - Labels must be globally unique within a pipeline
V. A quick tour of a messy Kedro project