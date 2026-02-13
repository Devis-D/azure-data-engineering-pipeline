# Architecture — Azure Data Engineering Pipeline

## Overview

This project implements a complete data engineering solution on Microsoft Azure, following the **Medallion Architecture** pattern to progressively refine data from raw ingestion to analytics-ready datasets.

## Data Flow

```
SQL Server (On-Prem)
    │
    ▼
Azure Data Factory ──── Azure Key Vault
    │                   (Secrets)
    ▼
ADLS Gen2: Bronze (Parquet)
    │
    ▼
Azure Databricks: Bronze → Silver Notebook
    │
    ▼
ADLS Gen2: Silver (Delta Lake)
    │
    ▼
Azure Databricks: Silver → Gold Notebook
    │
    ▼
ADLS Gen2: Gold (Delta Lake)
    │
    ▼
Azure Synapse Analytics (Serverless SQL)
    │
    ▼
Power BI (KPI Dashboard)
```

## Layer Descriptions

### Bronze Layer (Raw Zone)
- **Format**: Apache Parquet
- **Container**: `bronze/SalesLT/`
- **Purpose**: Exact copy of source data — no transformations applied
- **Tables**: 10 tables from the SalesLT schema
- **Ingestion**: ADF Copy Activity (SQL Server → ADLS Gen2)

### Silver Layer (Cleansed Zone)
- **Format**: Delta Lake
- **Container**: `silver/SalesLT/`
- **Purpose**: Cleaned, standardized, and deduplicated data
- **Transformations**:
  - Column renaming (CamelCase → snake_case)
  - Data type standardization
  - Null value handling
  - Duplicate removal
  - Date format normalization

### Gold Layer (Curated Zone)
- **Format**: Delta Lake
- **Container**: `gold/SalesLT/`
- **Purpose**: Business-ready, aggregated data optimized for analytics
- **Transformations**:
  - Business logic application
  - Gender-based analytics preparation
  - Revenue and sales aggregations
  - Dimension and fact table creation

## Orchestration

Azure Data Factory orchestrates the entire pipeline with four sequential activities:

1. **GetTables** (Lookup) → Discovers source tables dynamically
2. **LoopTables** (ForEach) → Copies each table to Bronze
3. **BronzeToSilver** (Databricks Notebook) → Cleanses data
4. **SilverToGold** (Databricks Notebook) → Applies business logic

A `daily_trigger` ensures the pipeline runs automatically every day.

## Security Architecture

- Azure Key Vault stores all sensitive credentials
- ADF accesses secrets at runtime via Key Vault linked service
- No connection strings or keys in pipeline definitions
- Managed identities where supported
