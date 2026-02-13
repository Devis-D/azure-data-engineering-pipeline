# Databricks Notebook: Silver → Gold

## Purpose
Reads cleansed Delta Lake tables from the **Silver** layer, applies business logic and aggregations, and writes analytics-ready datasets to the **Gold** layer as **Delta Lake** tables.

## Input / Output

| | Details |
|--|---------|
| **Source** | `abfss://silver@intechdataproject.dfs.core.windows.net/SalesLT/` |
| **Sink** | `abfss://gold@intechdataproject.dfs.core.windows.net/SalesLT/` |
| **Input Format** | Delta Lake |
| **Output Format** | Delta Lake |

## Tables Processed

All 10 SalesLT tables, transformed into analytics-ready datasets.

## Transformations Applied

### 1. Business Logic
- Customer gender classification and enrichment
- Order status categorization
- Product pricing tier assignment

### 2. Aggregations
- Revenue calculations by customer, product, and category
- Sales quantity summaries
- Customer demographic breakdowns

### 3. Data Enrichment
- Join customer data with address information
- Combine product details with categories and descriptions
- Merge sales order headers with line-item details

### 4. Analytics Optimization
- Create denormalized tables optimized for dashboard queries
- Pre-compute common aggregations (total revenue, product counts)
- Build gender-based analytics dimensions

## Business KPIs Supported
- Total Sales Revenue
- Total Products Sold
- Sales by Gender
- Sales by Product Category
- Customer Gender Distribution
- Date-based trend analysis

## Technology
- **Runtime**: Azure Databricks
- **Language**: PySpark
- **Triggered by**: Azure Data Factory (SilverToGold activity)
