# Databricks Medallion Architecture

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

Raw XML → Bronze (raw + validated)
↓
Silver (cleaned + enriched)
↓
Gold (denormalised + aggregated analytics)


---

## Bronze Layer

### Ingestion

- Read `Posts.xml` and `Users.xml` from volume
- Declarative transformations written in PySpark
- Schema defined using Databricks **Genie** for accurate struct inference  
  (avoids `inferSchema`, which can be slow or error-prone for XML)

---

### Bronze Data Quality (DQX)

Data quality checks implemented using the Databricks DQX framework.

#### Error Criticality Rules (Quarantine on Failure)

If these fail, records are split into a quarantined dataframe:

- `Id` must **NOT** be NULL  
- `CreationDate` must **NOT** be NULL  
- `CreationDate` must **NOT** be in the future  

Invalid records are separated for manual review.

---

#### Warning Criticality Rule

- `PostTypeId` must contain allowed values  

This is marked as **warning**, as new `PostTypeId` values may legitimately appear from the upstream source.

---

### Bronze Output

- Valid records → Bronze Delta tables  
- Invalid records → Quarantine dataframe/table  

---

## Silver Layer

The Silver layer applies business transformations to clean and enrich Bronze data.

### Transformations Applied

#### 1. Tag Normalisation
- Split `Tags` column by `|`
- Store as Python list (Spark array type) in a new column `TagsArray`

#### 2. Post Type Mapping
- Map `PostTypeId` to descriptive string values
- Create new column: `PostType`

---

### Optimisations

#### Full Refresh Logic
- Performs full load only if the target table does not exist.

#### Incremental Upsert (Delta Lake)

Implements incremental processing using `updated_at`:

1. Identify max `updated_at` in existing Silver table  
2. Filter incoming Bronze records where:


3. Perform Delta `MERGE` (upsert)

This reduces write volume and improves performance.

---

## Gold Layer

### 1. Denormalised Analytical Table

Creates a single wide table by joining:

- Posts
- Users

This produces a query-optimised dataset for reporting and analytics.

---

### 2. Tag Aggregation Table

Creates an aggregated Gold table:

- Explodes `TagsArray`
- Groups by tag
- Counts number of **distinct posts** per tag

**Result Schema:**


This enables analysis of tag popularity across posts.

---

## Key Concepts Demonstrated

- Medallion Architecture implementation
- Declarative Spark transformations
- Built-in schema struct generation
- Data quality enforcement with quarantine pattern
- Delta Lake incremental upserts
- Gold-layer denormalisation
- Aggregation modelling for analytics
- Governance-ready design

---

## Governance & Reliability Features

- Data lineage via Unity Catalog
- Delta Lake versioning & time travel
- Audit logging
- Quarantine pattern for invalid records
- Incremental data processing

---
