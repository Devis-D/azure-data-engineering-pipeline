# Databricks Notebook: Bronze → Silver

## Purpose
Reads raw Parquet files from the **Bronze** layer and applies data cleansing transformations, writing the output as **Delta Lake** tables to the **Silver** layer.

## Input / Output

| | Details |
|--|---------|
| **Source** | `abfss://bronze@intechdataproject.dfs.core.windows.net/SalesLT/` |
| **Sink** | `abfss://silver@intechdataproject.dfs.core.windows.net/SalesLT/` |
| **Input Format** | Apache Parquet |
| **Output Format** | Delta Lake |

## Tables Processed

All 10 SalesLT tables:
- Address, Customer, CustomerAddress
- Product, ProductCategory, ProductDescription
- ProductModel, ProductModelProductDescription
- SalesOrderDetail, SalesOrderHeader

## Transformations Applied

### 1. Schema Standardization
- Column renaming from CamelCase to snake_case convention
- Consistent naming across all tables

### 2. Data Type Casting
- Ensure numeric fields (prices, quantities) are proper decimal/integer types
- Standardize date columns to `timestamp` format
- Cast string fields to appropriate lengths

### 3. Null Handling
- Replace null values with appropriate defaults where applicable
- Flag records with critical null values for data quality monitoring

### 4. Deduplication
- Remove duplicate records based on primary key columns
- Maintain most recent record where duplicates exist

### 5. Date Formatting
- Standardize all date/datetime columns to ISO 8601 format
- Extract date components (year, month, day) for partitioning

## Technology
- **Runtime**: Azure Databricks
- **Language**: PySpark
- **Triggered by**: Azure Data Factory (BronzeToSilver activity)
