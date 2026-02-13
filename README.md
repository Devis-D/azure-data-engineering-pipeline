# Azure End-to-End Data Engineering Pipeline
### Customer Demographics & Sales Analytics

![Azure](https://img.shields.io/badge/Microsoft%20Azure-0078D4?logo=microsoftazure&logoColor=white)
![ADF](https://img.shields.io/badge/Azure%20Data%20Factory-0078D4?logo=azure-data-factory&logoColor=white)
![Databricks](https://img.shields.io/badge/Azure%20Databricks-FF3621?logo=databricks&logoColor=white)
![Synapse](https://img.shields.io/badge/Azure%20Synapse-0078D4?logo=microsoftazure&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?logo=powerbi&logoColor=black)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-003366?logo=delta&logoColor=white)

## Business Problem

A company identified a critical gap in understanding its customer demographics  specifically the gender distribution within its customer base and how it influences product purchases. With a significant volume of customer and sales data stored in an on-premises SQL Server database, key stakeholders requested a comprehensive KPI dashboard that provides:
- Insights into sales by gender and product category
- Total products sold and **total sales revenue
- A clear gender split among customers
- Ability to filter by product category, gender, and date

##  Solution Overview
Designed and built a fully automated end-to-end data pipeline on Microsoft Azure following the **Medallion Architecture (Bronze → Silver → Gold)**. The pipeline extracts data from an on-premises SQL Server database, transforms it through progressively refined layers, and delivers a custom-built Power BI dashboard — scheduled to run automatically every day, ensuring stakeholders always have access to up-to-date and accurate data.

##  Architecture


┌──────────────┐     ┌─────────────────────┐     ┌─────────────────────────────────────┐
│  On-Premises │     │   Azure Data Factory │     │   Azure Data Lake Storage Gen2      │
│  SQL Server  │────▶│   (Orchestration)    │────▶│                                     │
│  (SalesLT)   │     │  • Lookup Activity   │     │  ┌─────────┐ ┌────────┐ ┌────────┐ │
└──────────────┘     │  • ForEach Loop      │     │  │ Bronze  │▶│ Silver │▶│  Gold  │ │
                     │  • Daily Trigger     │     │  │(Parquet)│ │(Delta) │ │(Delta) │ │
                     └─────────┬────────────┘     │  └─────────┘ └────────┘ └────────┘ │
                               │                  └────────────────┬──────────────────-─┘
                     ┌─────────▼────────────┐                     │
                     │   Azure Key Vault    │     ┌───────────────▼───────────────────┐
                     │  (Secret Management) │     │     Azure Databricks              │
                     └──────────────────────┘     │  • Bronze → Silver Notebook       │
                                                  │  • Silver → Gold Notebook         │
                     ┌──────────────────────┐     └───────────────────────────────────┘
                     │      Power BI        │                     │
                     │  (KPI Dashboard)     │◀────┌───────────────▼───────────────────┐
                     └──────────────────────┘     │   Azure Synapse Analytics         │
                                                  │  (Serverless SQL Pools)           │
                                                   └───────────────────────────────────┘


##  Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Data Source** | SQL Server (On-Premises) | AdventureWorksLT database — SalesLT schema |
| **Ingestion & Orchestration** | Azure Data Factory | Dynamic pipeline with Lookup + ForEach, daily trigger |
| **Storage** | Azure Data Lake Storage Gen2 | Medallion Architecture (Bronze / Silver / Gold containers) |
| **Transformation** | Azure Databricks (PySpark) | Data cleansing, enrichment, and business logic |
| **Data Format** | Parquet → Delta Lake | Raw ingestion (Parquet), refined layers (Delta Lake) |
| **Analytics** | Azure Synapse Analytics | Serverless SQL pools for ad-hoc querying |
| **Visualization** | Power BI | Interactive KPI dashboards |
| **Security** | Azure Key Vault | Credential and secret management |

##  Data Pipeline Details

### 1. Ingestion — Azure Data Factory

The ADF pipeline `copy_all_tables` dynamically ingests all tables from the source database:

| Activity | Type | Description |
|----------|-----|-------------|
| GetTables | Lookup | Queries sys.tables joined with sys.schemas to auto-discover all tables in the SalesLT schema |
| LoopTables | ForEach | Iterates through each discovered table and copies data to ADLS Gen2 as Parquet files |
| BronzeToSilver | Databricks Notebook | Triggers the Bronze → Silver transformation notebook |
| SilverToGold | Databricks Notebook | Triggers the Silver → Gold transformation notebook |

Scheduling: Automated via a daily_trigger for continuous data freshness.

### 2. Storage — Azure Data Lake Storage Gen2

Data is organized in three containers following the Medallion Architecture:

| Layer | Container | Format | Description |
|-------|-----------|--------|-------------|
| **Bronze** | `bronze/` | Parquet | Raw data  exact copy from source, 10 tables |
| **Silver** | `silver/` | Delta Lake | Cleaned data  type casting, deduplication, null handling |
| **Gold** | `gold/` | Delta Lake | Curated data  business logic, aggregations, analytics-ready |

### 3. Tables Processed (10 Total)

| # | Table Name | Description |
|---|-----------|-------------|
| 1 | `Address` | Customer and company addresses |
| 2 | `Customer` | Customer demographics and contact info |
| 3 | `CustomerAddress` | Customer-to-address mapping |
| 4 | `Product` | Product catalog with pricing |
| 5 | `ProductCategory` | Product category hierarchy |
| 6 | `ProductDescription` | Product descriptions and details |
| 7 | `ProductModel` | Product model information |
| 8 | `ProductModelProductDescription` | Model-to-description mapping |
| 9 | `SalesOrderDetail` | Line-item sales order details |
| 10 | `SalesOrderHeader` | Sales order headers with dates and totals |

### 4. Transformation — Azure Databricks

Two PySpark notebooks handle the data transformation:

- **Bronze → Silver**: Data cleansing, column renaming (CamelCase to snake_case), data type casting, null handling, deduplication, and date formatting
- **Silver → Gold**: Business logic application, aggregations, gender-based analytics, revenue calculations, and creation of analytics-ready dimension and fact tables

### 5. Analytics — Azure Synapse Analytics

Serverless SQL pools provide:
- External tables over Gold-layer Delta files
- Ad-hoc SQL querying without data movement
- Views optimized for Power BI consumption

### 6. Visualization — Power BI

Interactive KPI dashboard featuring:
- **Total Sales Revenue** — aggregated across all orders
- **Total Products Sold** — count of distinct products
- **Sales by Gender** — revenue and quantity breakdown by customer gender
- **Sales by Product Category** — category-level performance metrics
- **Customer Gender Distribution** — demographic split visualization
- **Date-based filtering** — interactive timeline for trend analysis
- **Cross-filters** — filter by product category, gender, and date range

##  Security
- **Azure Key Vault** manages all secrets (SQL Server connection strings, storage keys, Databricks tokens)
- Integrated with ADF Linked Services for secure credential retrieval at runtime
- No credentials stored in pipeline code or configuration

##  Project Structure
azure-data-engineering-pipeline/
├── README.md                          # Project documentation
├── adf/
│   ├── pipeline/
│   │   └── copy_all_tables.json       # ADF pipeline definition
│   ├── linkedService/
│   │   └── linked_services.json       # Linked service configurations
│   └── dataset/
│       └── datasets.json              # Dataset definitions
├── databricks/
│   ├── bronze_to_silver.md            # Bronze → Silver notebook overview
│   └── silver_to_gold.md              # Silver → Gold notebook overview
└── docs/
    └── architecture.md                # Detailed architecture documentation
## Key Highlights

- **Fully Automated** — Daily scheduled trigger ensures data is always current
- **Dynamic Ingestion** — No hardcoded table names; auto-discovers all tables via metadata queries
- **Medallion Architecture** — Progressive data quality refinement (Bronze → Silver → Gold)
-  **Delta Lake** — ACID transactions, time travel, and schema evolution in Silver/Gold layers
-  **Scalable** — Adding new source tables requires zero pipeline changes
-  **Secure** — All credentials managed through Azure Key Vault
-  **Cost-Optimized** — Serverless compute in Synapse (pay-per-query)

##  Azure Resources

| Resource | Type | Region |
|----------|------|--------|
| `INTECH-data-project-ADF` | Azure Data Factory | Germany West Central |
| `intechdataproject` | Storage Account (ADLS Gen2) | Germany West Central |
| `databricks-intech-project` | Databricks Workspace | Germany West Central |
| `intech-project` | Synapse Analytics Workspace | Germany West Central |
| `INTECH-data-project-ADF` | Key Vault | Germany West Central |
| `intechdatasynapse` | Storage Account (Synapse) | Germany West Central |

## Author

Devis Doci
- GitHub: [@devisdoci00-lgtm](https://github.com/devisdoci00-lgtm)
