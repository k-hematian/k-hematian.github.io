---
title: "The Architecture of Evolution: Why the Lakehouse is Inevitable"
date: 2026-01-01
isStarred: false
description: "Great architecture isn't about chasing tools. A deep dive into the CIDR 2021 paper on why the 'Two-Tier' architecture is obsolete and how the Lakehouse works under the hood."
tags:
  - data engineering
  - architecture
  - lakehouse
  - system design
  - paper review
---

## The Structural Bottleneck
In [an earlier post](/posts/2025-12-27-the-lakehouse-bridging-the-gap), I touched on why Data Lakes emerged. But to understand where the industry is going next, we need to understand the history of how we got here.

Great architecture isn't about chasing new tools; it's about recognizing the structural bottlenecks that force systems to evolve.

The paper "Lakehouse: A New Generation of Open Platforms" (CIDR 2021) is a brilliant breakdown of this evolution. It maps the shift from Gen 1 (Warehouses) to Gen 2 (Lakes + Warehouses) and explains why the industry is inevitably moving to Gen 3 (Lakehouse).

The paper argues that the standard "Two-Tier" architecture (gluing a Warehouse to a Lake) is obsolete. It creates Data Swamps, high TCO, and forces engineers to maintain complex Lambda Architectures just to handle streaming and batch together.

For those interested in system internals, the paper details how they achieve Warehouse performance on top of open object storage (S3/ADLS) using:

* **Metadata Layers:** Implementing ACID transactions and Snapshot Isolation directly on Parquet/ORC files.
* **Data Skipping:** Using Z-Order Space-Filling Curves and Bloom Filters to minimize I/O without proprietary indexing.
* **Unified Engines:** Decoupling compute from storage to run SQL and ML on the exact same dataset without copying.

I’ve attached the original paper below[^1], along with my personal summarized notes (a cheat sheet of the key diagrams and concepts) to save you some time.

---

## Summary Notes: Lakehouse Architecture
This document reviews the seminal paper *"Lakehouse: A New Generation of Open Platforms that Unify Data Warehousing and Advanced Analytics"* (CIDR 2021).

In this paper, the team at **Databricks** (including CEO Ali Ghodsi) forecasts the industry's shift toward a unified data management architecture. They argue that the "Two-Tier" architecture (Data Lakes + Data Warehouses) is becoming obsolete due to cost, complexity, and staleness. They propose the **Lakehouse Architecture**: a third-generation platform that combines the low-cost storage of Data Lakes with the performance and ACID guarantees of Data Warehouses.

### History
![image](/images/lakehouse-paper-generations.png)

#### First Generation Platforms
The history of storing data in one place dates back to when systems scaled and data got distributed. Then some business requirements became prominent:
* Access to data for Business Intelligence (making decisions based on data).
* Call centers need to access integrated data to answer customers.

To answer this demand, the first generation of Data Warehouse systems was provided. In these systems, all data from operational databases was gathered into one data warehouse to satisfy these demands. They were **Schema-on-write** and had negligible time drift with operational databases.

About one decade ago (early 2010s), this architecture faced some challenges:
1. **Coupled Resources:** Typically the storage and compute resources were coupled to an **On-Premise** appliance. So companies had to provision resources and pay for peak usage for the entire dataset, which was growing rapidly.
2. **Data Variety:** Unstructured datasets were growing, and data warehouses could not store or query formats such as video, audio, and text documents efficiently.

**Advantages:**
* **Schema-on-write** for data integrity and cleanliness.
* Simple and single **Extract-Transform-Load (ETL)** step.

**Disadvantages:**
* **Schema-on-write** decreases **Agility** and **Evolvability**.
* High cost due to coupled compute and storage.

#### Current Two-tier Architecture
To solve these challenges, the second generation data analytics platforms started offloading all raw data into **Data Lakes**: low-cost storage systems with a file API that hold data in generic and usually open file formats, such as **Apache Parquet** and **Optimized Row Columnar (ORC)**. This approach was started by the **Apache Hadoop** movement.

This architecture, by implementing **Schema-on-read** and leveraging **Hadoop File System (HDFS)**, enabled the agility of storing *any* data at low cost. However, this often led to the **"Data Swamp"** problem, where reliability and quality were compromised downstream. This architecture also includes **Extract-Transform-Load (ETL)** to load a subset of data into a downstream **Data Warehouse** for the most important decision support and BI applications.

From 2015 onward, cloud data lakes such as **AWS S3**, **Azure Data Lake Storage (ADLS)**, and **Google Cloud Storage (GCS)** started replacing **Hadoop File System (HDFS)**.

* **Decoupled Compute and Storage:** Cloud lakes allowed storage to scale infinitely and cheaply, independent of compute engines.
* Superior **Data Durability** (often > 10 nines) and **Geo Replication**.

**Advantages:**
* **Schema-on-read** for **Evolvability** and **Agility**.
* Cheap storage to be able to keep decades of data.
* Use of **Open Formats** to be directly accessible for a wide range of other analytics engines such as **Machine Learning** systems.

**Disadvantages:**
* **Reliability:** As there are two steps in this architecture—first ingest data into a **Data Lake** and then **ETL** to a data warehouse—the chance of inconsistency between these sources is higher and demands continuous engineering. Each step of ETL also increases the risk of bugs due to differences between the datalake and warehouse engines.
* **Data Staleness:** Data in the warehouse might be stale compared to that of the data lake, with new data frequently taking days to load.
* **Limited support for advanced analytics:** Businesses want to ask predictive questions using their warehousing data, such as "which customers should I offer discounts to?", but currently there is no compatibility between **Machine Learning** systems such as **TensorFlow**, **PyTorch**, and **XGBoost** and Data Warehouses. Unlike BI queries which extract a small amount of data, these systems need to process large datasets using complex non-SQL code. Reading data via **ODBC** and **JDBC** is inefficient and there is no direct access to data warehouse data files. There are two solutions:
    1. Add another ETL step.
    2. Use the data lake directly at the cost of losing **ACID Transactions**, data versioning, and indexing.
* **Total cost of ownership:** Extra ETL steps, more storage cost due to copying subsets of data from the data lake to the data warehouse, and the extra cost of migrating data from the warehouse to other systems.

> **Fact: Data Staleness Report**
>
> According to a survey by [Dimensional Research and Fivetran](https://fivetran.com/blog/analyst-survey), 86% of analysts use out-of-date data and 62% report waiting on engineering resources numerous times per month.

---

### Solutions: The Lakehouse Architecture
The current status is the **two-tier** architecture and its challenges. Removing the data lake and storing all data into the data warehouse is not a solution because:
1. It does not support data formats such as video/audio/text.
2. It does not provide its data efficiently for **Machine Learning** use cases.

The paper suggests merging these **two tiers** into **one tier** by keeping the best of both:
* SQL Performance, Table versioning, indexing, caching, and optimization of Data Warehouses.
* Open format, cheap storage, evolvability, and agility of Data Lakes.

![image](/images/lakehouse-paper-architecture.png)

The paper suggests it's possible and provides an architecture for this matter:

1. Using **Apache Parquet** or similar open format data files in **Object Storage** as the storage layer.
2. Add a transactional **metadata layer** such as **Apache Iceberg**, **Databricks Delta Lake**, and **Apache Hudi** over storage to improve query performance while adding the following features:
    1. Transactions (**ACID Transactions**).
    2. **Zero-Copy Cloning Technique**.
    3. Time-travel to past versions of a table.
    4. Quality enforcement by enforcing schema to prevent writing invalid data (solving the Data Swamp issue).
3. Computation engines with SQL API such as **Apache Spark** to query the open format data efficiently with help of the provided metadata.
4. Computation engines with **Declarative DataFrame APIs** for **Machine Learning** usages.

**Key Architectural Insights:**
* **Unified Streaming and Batch:** The Lakehouse solves the "Lambda Architecture" problem. Tables can be treated as a continuous stream of changes, allowing real-time ingestion and batch querying on the same data structures simultaneously.
* **Diverse Workloads:** Data Scientists (who need raw data) and BI Analysts (who need curated data) access the *same* single source of truth without moving data around.

There are decisions which can be changed in this architecture by time and maturity of the industry:
1. What **Open Format** is suitable? Is it better to innovate a new open format to increase performance (at the cost of compatibility)?
    * **Databricks Delta Lake** decided to use **Apache Parquet** as it's highly adapted in query engines and **Machine Learning** systems.
2. **Where to save metadata?** Storing in a separate DBMS (RAM or SSD) to increase throughput/reduce latency, or alongside the data?
    * In **Databricks Delta Lake**, they decided to store the metadata layer into the same **Object Storage** as the other data at rest to decrease maintenance costs and leverage the massive scalability of cloud object stores.

### Optimization to Reach SQL Performance
To reach SQL performance, there are some places to optimize:
* **Caching:** By storing highly used data in fast SSDs or RAM, it's possible to use all "closed-world" data warehouse engines and reshape data to some form that is simpler to query.
* **Auxiliary Data:** Storing auxiliary data to make queries easier. **Databricks Delta Lake** maintains:
    * Column min-max statistics for each data file in the table within the same Parquet file used to store the transaction log.
    * **Bloom Filter** based index.
* **Data Layout:** Use an optimized data format or optimize the existing data formats as much as possible. **Databricks Delta Lake** supports ordering records using individual dimensions or space-filling curves such as **Z-Order Curve** and Hilbert curves to provide locality across multiple dimensions.

These optimizations improve:
* In typical workloads, most queries tend to be concentrated against a "hot" subset of the data which can be easily cached and queried with competitive performance.
* For "cold" data, the combination of data layout and auxiliary data allows a Lakehouse system to minimize I/O the same way a closed-world proprietary data warehouse would, despite running against a standard open file format.

### Performance Result
![image](/images/lakehouse-paper-tpc-ds.png)

> At Databricks, we combined these three Lakehouse optimizations with a new C++ execution engine for Apache Spark called Delta Engine. To evaluate the feasibility of the Lakehouse architecture, Figure 3 compares Delta Engine on TPC-DS at scale factor 30,000 with four widely used cloud data warehouses (from cloud providers as well as third-party companies that run over public clouds), using comparable clusters on AWS, Azure and Google Cloud with 960 vCPUs each and local SSD storage We report the time to run all 99 queries as well as the total cost for customers in each service's pricing model (Databricks lets users choose spot and on-demand instances, so we show both). Delta Engine provides comparable or better performance than these systems at a lower price point.


[^1]: [Lakehouse: A New Generation of Open Platforms (CIDR 2021)](https://www.cidrdb.org/cidr2021/papers/cidr2021_paper17.pdf)