# Pipeline Architecture – Shopware Enterprise Data Lakehouse

## Overview

This document outlines the technical architecture of the enterprise-grade data pipeline built for Shopware. The architecture integrates both batch and streaming data into a secure, governed **Lakehouse** powered by **Apache Iceberg**, utilizing the **Medallion Architecture** to deliver reliable, scalable, and actionable analytics.

---

## Architecture Style: Medallion Lakehouse (Bronze → Silver → Gold)

```

```
              +----------------------+
              |    Data Producers    |
              +----------------------+
                 | Batch & Streaming
                 v
            +--------------------+
            |     Bronze Layer   |  <-- Raw data in S3
            | (Partitioned S3 w/ |
            | Apache Iceberg)    |
            +--------------------+
                    |
                    v
            +--------------------+
            |     Silver Layer   |  <-- Cleaned & conformed
            |  (Glue-transformed |
            |   Iceberg tables)  |
            +--------------------+
                    |
                    v
            +--------------------+
            |     Gold Layer     |  <-- Aggregated & enriched
            | (BI-ready metrics, |
            |  Iceberg format)   |
            +--------------------+
                    |
                    v
     +------------------------------+
     | Redshift Spectrum + Dashboards|
     +------------------------------+
```

```

---

## Key Components

### 🔁 Data Ingestion

- **Streaming Sources**: Web Logs & Customer Interactions
  - Ingested via custom **Lambda/Kinesis → S3** scripts (Ebenezer)
  - Landed in the **Bronze Layer** (raw bucket with Apache Iceberg)
- **Batch Sources**: POS & Inventory Management
  - Periodically synced to **Bronze Layer S3** via AWS Glue or Lambda

---

### 🔄 ETL & Transformation (Bronze → Silver → Gold)

- **Bronze Layer**: Immutable raw data
  - Format: **Apache Iceberg** tables (partitioned by date/source)
  - Tools: Lambda, Kinesis, direct S3 writes
- **Silver Layer**: Validated & structured datasets
  - Format: Iceberg tables optimized for query
  - ETL Jobs: **AWS Glue PySpark scripts** (Brempong)
  - Tasks: Schema enforcement, null checks, deduplication, enrichment
- **Gold Layer**: Aggregated KPIs and team-specific views
  - KPI-aligned Iceberg tables
  - Loaded via Glue into partitioned tables ready for Redshift Spectrum and dashboards

---

### 🏢 Data Storage & Access

- **S3 Buckets**
  - `/bronze/`, `/silver/`, `/gold/` - Medallion layers
  - `/rejected/` - Invalid records
- **Apache Iceberg Tables**
  - Transactional tables per source and layer
  - Stored on S3, catalogued via Glue Data Catalog
- **Redshift Spectrum**
  - External schema pointing to Gold Iceberg tables
  - Used by dashboards and ad-hoc SQL tools
- **Amazon QuickSight**
  - Visualization layer for dashboards and KPI boards

---

### 🔐 Governance, Security & Access Control

- **Lake Formation**: Central RBAC control
  - Fine-grained permissions on tables, columns, and rows
  - Defined access policies by team (Sales, Ops, Marketing, Support)
- **Encryption**:
  - S3: SSE-S3 / SSE-KMS for data at rest
  - In transit: HTTPS and VPC endpoints
- **Audit Logging**: CloudTrail + Lake Formation logs

---

## Processing Flow

1. **Ingestion**
   - Streaming: Data arrives via Lambda → S3 → Bronze
   - Batch: Loaded via Glue job on schedule
2. **Bronze → Silver**
   - Cleansing, type casting, validation
   - Deduplication and timestamp formatting
3. **Silver → Gold**
   - KPI aggregation
   - Business logic application
   - Optimization for downstream access
4. **Query & Visualization**
   - Data exposed to Redshift Spectrum and QuickSight

---

## Future Enhancements

- **Schema Evolution**: Enable automatic version tracking with Iceberg snapshots
- **Data Lineage**: Integrate AWS Glue DataBrew or Amundsen for metadata visibility
- **CI/CD**: Automate pipeline testing and deployment with GitHub Actions

---

**Maintainer**: Ebenezer Quayson  
```


