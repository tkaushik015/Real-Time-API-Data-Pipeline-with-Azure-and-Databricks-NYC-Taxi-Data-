# NYC-TAXI-DE-Project

# 🚀 Azure End-to-End Data Engineering Pipeline (From Zero to Pro)

🔗 **GitHub Repository:** https://github.com/yourusername/azure-end-to-end-data-engineering

---

## 📚 Table of Contents
1. [Project Overview](#project-overview)  
2. [🏗️ Architecture](#architecture)  
3. [💾 Data Sources](#data-sources)  
4. [⚙️ Key Features & Concepts](#key-features--concepts)  
5. [🔧 Technology Stack](#technology-stack)  
6. [🚀 Getting Started](#getting-started)  
   - 🛠️ [Prerequisites](#prerequisites)  
   - 📥 [Clone the Repo](#clone-the-repo)  
   - ☁️ [Azure Provisioning](#azure-provisioning)  
   - ⚙️ [Configuration](#configuration)  
   - ▶️ [Deployment & Execution](#deployment--execution)  
7. [📂 Folder Structure](#folder-structure)  
8. [🧠 Key Learnings & Challenges](#key-learnings--challenges)  
9. [🔮 Future Improvements](#future-improvements)  
10. [📖 References](#references)  

---

## 📚 Project Overview
A production-grade data pipeline on Microsoft Azure that:
- **Ingests** monthly NYC “Green Taxi” Parquet files directly from the NYC Open Data API.  
- **Processes** data through Bronze → Silver → Gold Delta Lake layers in Azure Databricks.  
- **Implements** dynamic orchestration with Azure Data Factory (ADF), SCD Type 2 history tracking, secure secrets via Key Vault, and parallel execution.  
- **Serves** curated fact & dimension tables to Power BI for interactive analytics.  

---

## 🏗️ Architecture
```text
            ┌────────────────────┐      ┌────────────────────┐
            │  NYC Open Data API │      │   Static Lookups    │
            │   (Green Taxi)     │      │  (zone_lookup.csv)  │
            └─────────┬──────────┘      └─────────┬───────────┘
                      │                           │
     ┌────────────────▼───────────────────────────▼────────────────┐
     │                    Azure Data Factory (ADF)                │
     │  • Lookup JSON config → ForEach(months)                    │
     │  • Copy Activities → Bronze zone in ADLS Gen2              │
     │  • Trigger Databricks notebooks via Notebook Activity      │
     └────────────────┬────────────────────────────────────────────┘
                      │
          ┌───────────▼───────────┐
          │     Bronze Zone       │
          │ ADLS Gen2 (raw Parquet│
          │    & lookup CSV)      │
          └───────────┬───────────┘
                      │
          ┌───────────▼───────────┐
          │     Silver Zone       │
          │ Databricks Notebooks  │
          │ • Schema Conformance  │
          │ • SCD Type 2 Merges   │
          └───────────┬───────────┘
                      │
          ┌───────────▼───────────┐
          │      Gold Zone        │
          │ Databricks Jobs       │
          │ • Fact & Dim Tables   │
          └───────────┬───────────┘
                      │
          ┌───────────▼───────────┐
          │     Power BI Live     │
          │   Connector to Gold   │
          └───────────────────────┘

💾 Data Sources
🟢 NYC Taxi “Green” API
Monthly Parquet files pulled via REST API.

📑 Zone Lookup CSV
Static reference mapping LocationID → Borough, Zone Name.

⚙️ Config JSON
Defines active months, file patterns, ADLS paths for dynamic ADF pipelines.

⚙️ Key Features & Concepts
Dynamic, Parameterized ADF Pipelines

ADF Lookup reads config/green_taxi_config.json.

ForEach iterates over each month, injecting parameters into Copy & Notebook activities.

Medallion Architecture (Bronze → Silver → Gold)

Bronze: Raw Parquet + CSV in ADLS Gen2.

Silver: PySpark in Databricks for schema standardization, surrogate keys.

Gold: Star-schema FactTrips & Dimensions for analytics.

SCD Type 2 History Tracking

sql
Copy
Edit
MERGE INTO silver.Trips AS tgt
USING updates AS src
  ON tgt.trip_id = src.trip_id AND tgt.is_current = true
WHEN MATCHED AND src.* <> tgt.* THEN
  UPDATE SET is_current = false, end_date = current_timestamp()
WHEN NOT MATCHED THEN
  INSERT *
Preserves change history with start_date, end_date, is_current.

Delta Lake Best Practices

Partition by month, payment_type.

Z-ORDER on high-cardinality columns.

Time Travel for point-in-time queries & recovery.

Secure Secret Management

Azure Key Vault stores storage & Databricks credentials.

ADF & Databricks access secrets via Managed Identities—no hard-coded keys.

Parallel Execution & Audit

ForEach batchCount = 6 for concurrent month processing.

Audit table in Azure SQL tracks pipeline run status & row counts.

Power BI Connectivity

Live connector to Gold Delta tables for sub-hourly dashboard refreshes.

🔧 Technology Stack
Layer	Tools & Services
Orchestration	Azure Data Factory
Storage	Azure Data Lake Storage Gen2
Compute	Azure Databricks (PySpark, Delta Lake)
Security	Azure Key Vault, Managed Identities
Visualization	Power BI
Source Control	Git & GitHub
API	NYC Open Data REST API

🚀 Getting Started
🛠️ Prerequisites
Azure subscription with:

Data Factory, Storage Account (ADLS Gen2), Databricks workspace, Key Vault

Local tools: Azure CLI, Databricks CLI, Git

📥 Clone the Repo
bash
Copy
Edit
git clone https://github.com/yourusername/azure-end-to-end-data-engineering.git
cd azure-end-to-end-data-engineering
☁️ Azure Provisioning
bash
Copy
Edit
# Create resource group
az group create --name rg-data-pipeline --location eastus

# Storage account & containers
az storage account create --name dldevelstorage --resource-group rg-data-pipeline --hierarchical-namespace true
az storage container create --name bronze --account-name dldevelstorage
az storage container create --name lookup --account-name dldevelstorage
az storage container create --name silver --account-name dldevelstorage
az storage container create --name gold --account-name dldevelstorage

# Key Vault & secrets
az keyvault create --name kv-data-pipeline --resource-group rg-data-pipeline
az keyvault secret set --vault-name kv-data-pipeline --name "StorageKey" --value "<storage-key>"
az keyvault secret set --vault-name kv-data-pipeline --name "DatabricksPAT" --value "<databricks-pat>"
⚙️ Configuration
Upload zone_lookup.csv to lookup/ container.

Place config/green_taxi_config.json in lookup/ on ADLS:

json
Copy
Edit
[
  { "month": "202201", "isActive": true },
  { "month": "202202", "isActive": true },
  …  
  { "month": "202212", "isActive": false }
]
▶️ Deployment & Execution
Import ADF Pipelines from pipelines/ADF_JSON_definitions/ into your Data Factory.

Configure ADF Linked Services to reference Key Vault secrets.

Trigger pipeline_green_taxi_master in ADF.

Monitor runs in ADF Monitor & view audit table in Azure SQL.

Validate Delta tables in Databricks:

sql
Copy
Edit
SELECT COUNT(*) FROM gold.factTrips;
DESCRIBE HISTORY silver.Trips;
Connect Power BI to Gold schema via Databricks connector.

📂 Folder Structure
pgsql
Copy
Edit
├── configs/
│   └── green_taxi_config.json
├── lookup/
│   └── zone_lookup.csv
├── notebooks/
│   ├── 01_ingest_bronze.py
│   ├── 02_transform_silver.py
│   └── 03_build_gold.py
├── pipelines/
│   └── ADF_JSON_definitions/
└── README.md
🧠 Key Learnings & Challenges
Dynamic Orchestration: JSON-driven pipelines eliminated manual edits for new months.

SCD Type 2 Merges: Mastered Delta Lake MERGE syntax for history tracking.

Parallelization Pitfalls: Removed sequential bottlenecks in ForEach & audit table auto-increment.

Security Best Practices: Centralized secrets in Key Vault; used Managed Identities.

🔮 Future Improvements
⚡ Integrate CDC streams via Event Hubs for near real-time ingest.

📊 Adopt Unity Catalog in Databricks for unified governance & lineage.

🔄 Implement CI/CD for ADF & notebooks via Azure DevOps or GitHub Actions.

✅ Add automated data quality checks with Deequ or Great Expectations.

📖 References
Azure Data Factory Documentation

Delta Lake Guide

Azure Databricks Best Practices

NYC Open Data API

makefile
Copy
Edit
::contentReference[oaicite:0]{index=0}
