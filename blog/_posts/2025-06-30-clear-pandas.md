---
#layout: post
title: "A Suggestion for Clear Pandas Implementation"
description: >
  Pandas implementation practices that advocates for transformation-centric design inducing high code readability and modularization.
author: author1
comments: true
---

**Code**: <a href="https://github.com/hovinh/clearpandas">Github</a> 

## Pandas Get Dirty Sometimes

`panda` is a powerful tool for working with tabular data, offering a wide range of built-in functions that make data transformation tasks straightforward.
However, I’ve noticed that as more tables are integrated and transformations accumulate, the code tends to get “dirty” fast: the logic becomes harder to follow, and it’s easy to overlook certain steps—especially when reusing the same transformation chains across similar tables.

It’s natural to wrap these transformations into functions to make them more manageable.
But I don’t think that’s the best long-term solution.

Ideally, I want to establish a systematic way of organizing the code before I even begin, so that the logic is structured, maintainable, and easy to read from the outset.
That’s the purpose of this blog.

In the sections that follow, I’ll outline how I translate data-related entity concepts into code, along with a brief guide on assembling them into a working pipeline.

- [The Concept](#the-concept)
- [The Implementation](#the-implementation)
  - [DataConnector](#dataconnector)
  - [Transform](#transform)

## The Concept

To me, a data pipeline is simple: it's a graph whose nodes are either `DataConnector` or `transform`.
A `DataConnector` handles reading from and writing to data sources, regardless of their origin. 
A `transform` handles the self-transform or multi-table interaction logic, for example an outer join.
These two entities communicate by accepting and returning `pandas.DataFrame` objects.
Together, they provide everything you need to build a functional `Pipeline`.

As more tables come into play, these entities need to be organized into groups. 
Each group should have a clear, descriptive name that reflects the transformation objective it performs.
As a starting point, I typically use the classic **ETL** (Extract, Transform, Load) structure to guide the grouping. 
If the logic becomes more complex, I introduce intermediate groups to maintain clarity and modularity.

**Fig. 1** summarizes the core concepts, their semantic interactions, and how they are grouped.

![Fig01](/assets/blog/2025-06-30/dataflow.png){:data-width="1440" data-height="836"}
Fig. 1. The concept of entities in a data pipeline.
{:.figure}

## The Implementation

### DataConnector

Developers can delegate and abstract away the task of establishing connections and querying from various data sources through a unified interface: `DataConnector`.

```python
class DataConnector:
    def __init__(self, source):
        self.source = source

    def get_data(self, query: str = None) -> pd.DataFrame:
        pass
    
    def write_data(self, pd_table: pd.DataFrame):
        pass
```

All concrete connectors adhere to this interface and implement their source-specific logic by inheriting from `DataConnector`.
While this example assumes a pure Python workflow, the same structure can be adapted for PySpark by replacing all instances of `pandas.DataFrame` with `pyspark.sql.DataFrame`.
Notably, the `duckdb` package enables SQL-query support as long as the loaded table is stored in a variable named `df`. 
This eliminates the need to come up with a meaningful table names in small utility functions - a convenience I strongly recommend applying later in `transform` as well. 

```python
class CSVConnector(DataConnector):
    def __init__(self, source):
        self.source = source

    def get_data(self, query: str = None)-> pd.DataFrame:
        df = pd.read_csv(self.source)
        if (query):
            con = duckdb.connect()
            df = con.execute(query).df()
        return df
    
    def write_data(self, df: pd.DataFrame):
        df.to_csv(self.source, index=False)
```

We can take the abstraction one step further by removing the need for developers to manually select the appropriate connector.
Instead, we delegate that responsibility to a `ConnectorFactory`:

```python
class ConnectorFactory:
    @staticmethod
    def create_connector(source: str) -> DataConnector:
        if source.endswith('.csv'):
            return CSVConnector(source)
        elif source.endswith('.xlsx'):
            return ExcelConnector(source)
        else:
            raise ValueError("Unsupported file format. Use .csv or .xlsx") 
```

With this setup, input/output operations can be cleanly abstracted using `get_data()` and `write_data()`, even as concise one-liners if desired:

```python
customer_connector = ConnectorFactory().create_connector('data/customers.csv')
customer_table = customer_connector.get_data("SELECT * FROM df WHERE c_acctbal > 200")

order_details_connector = ConnectorFactory().create_connector('data/urgent_order_details.csv')
order_details_connector.write_data(data['urgent_order_details']) 
```

## Transform


`pandas` lends a useful way to 

chain
structure
naming
df 

🧠 Conceptual Difference
Feature	apply()	pipe()
Function scope	Often row/column-wise (axis)	Always passes the entire DataFrame
Use case	Per-row/column ops, aggregation	Whole-frame transformation

| Use `apply()` when...                                         | Use `pipe()` when...                               |
| ------------------------------------------------------------- | -------------------------------------------------- |
| You need row-wise or column-wise operations (`axis=1/0`)      | You're applying a function to the whole DataFrame  |
| You're doing aggregations like `.mean()` or `.sum()` row-wise | You're composing transformations in a method chain |
| You need more control over individual rows or columns         | You want clean, modular pipelines     

two table
```
📦 my_project 
├── 📁 src
|    ├── 📁 transform 
│    ├── 📄 base.py 
│    └── 📄 customers.py 
│    └── 📄 orders.py 
...
```

```python
processed_customer_table = (
            customer_table
            .pipe(transform.customers.round_up_acctbal)
            .pipe(transform.customers.filter_customers, max_custkey=1000)
            )
```


## The Assemble

code structure.

## Data Connector


## Transform


## Exceptions
- load minor talbe and not used, put in transform for simple sake

## Concept of Transformation


## A

function naming

Typical "treatment" for tabular data in python is the employing of `pandas` package.
Pandas is widely used 
It's common p
 package in Python is a powerful, open-source library designed for data manipulation and analysis. It provides fast, flexible, and expressive data structures that make working with “relational” or “labeled” data both easy and intuitive

 


 no inplace arguemtn
Pandas is an excellent tool

Tabular data receives finetuning from gold to silver. 

Working with tabular a
Data regression.

Raw tabular data type to silver.

no standard way to structure transformation.

A typical Machine Learning project may have
but when data needs further processing.

A common instance 

df multiple times, and come . come up with descriptive name, the code longer
chain transformation, but low level and hard to make sense
code duplication

besides, data readdata query and write


OOP
abstraction

abstraction for the flexible step of transformation in pandas

## Design



## Implement


## Practice

code structuring
in certain case, pipeline is optional

lego block
naming is verb

duckdb to enable SQL

recipe: by a chain of transforamtion

This pipeline design is structured around four core entities that work together to manage data flow and transformation:
- Connector: Responsible for reading data from and writing data to external sources. It acts as the interface between the pipeline and the outside world (e.g., databases, files, APIs).
- Transform: Handles intermediate data processing. Each transform takes in data, applies a specific transformation logic, and outputs the modified data for the next step.
- Datafram: Serves as the data container passed between connectors and transforms. It represents the input and output at each stage of the pipeline, ensuring consistency and traceability of the data.
- Pipeline: Orchestrates the sequence of connectors and transforms. It defines the overall workflow by chaining these components into a cohesive process, enabling modular and reusable data operations.
This design promotes clarity, modularity, and flexibility, making it easy to build, maintain, and extend data workflows.

- Factory for the kind of connector needed, so you can abstract away or make method consistent. Excel, Parquet.

1. Source (data retrieval / extraction)
2. Transformation (filter, join, derive, aggregate)
3. Output (store / visualize / export)

what happen if there are duplicate methods between dataset
 Avoid Using In-Place Modification
df variable is within the method itself
think of different name, long and hard to read



             |

organization into transform
a hierachy
ETL: makes it easy to know where to find the table. a self transform is Extract

conceptual/semantic to physical

Join: name of the new table

data lineage is clear since change name or show up once
assume 1 table out of these operations
DAG



can see the code example at Github



---