# StackExchange Medallion Architecture Project (Databricks)

## Overview

This project implements a **Medallion Architecture (Bronze → Silver → Gold)** pipeline in Databricks using the StackExchange dataset.

It demonstrates:

- Declarative Spark transformations (PySpark)
- Built-in schema detection
- Data quality enforcement using DQX
- Incremental upserts with Delta Lake
- Governance and auditability best practices
- Gold-layer analytical modelling

---

## Dataset

**Source:** StackExchange Data Dump  
https://archive.org/download/stackexchange  

Files used:
- `Posts.xml`
- `Users.xml`

The XML files are read from a Databricks volume and processed through Bronze, Silver, and Gold layers.

---

## Architecture

