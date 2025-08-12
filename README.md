
# Azure End-to-End Data Engineering Project 🚀

## 📌 Project Overview
This project demonstrates the design and implementation of a **real-world data engineering pipeline** using **Microsoft Azure** services.  
It follows the **Medallion Architecture (Bronze → Silver → Gold)** for structured, scalable, and analytics-ready data processing.

The project ingests raw CSV data from Azure Data Lake (Bronze), transforms it with PySpark in Azure Databricks (Silver), and loads it into Azure Synapse Analytics (Gold) for Power BI reporting.

Data Set: https://www.kaggle.com/datasets/ukveteran/adventure-works
---

## 🏗 Architecture


**Workflow:**
1. **API Source / External Dataset** → Ingested to Bronze Layer via **Azure Data Factory**
2. **Bronze Layer (Raw)** → Stored in **Azure Data Lake Gen2**
3. **Silver Layer (Cleaned & Enriched)** → Processed in **Azure Databricks (PySpark)**
4. **Gold Layer (Analytics-Ready)** → Created in **Azure Synapse Analytics** using External Tables & Views
5. **Power BI** → Connected to Synapse for reporting

---

## 🛠 Tech Stack
- **Azure Data Factory** – Orchestration & ETL Pipelines
- **Azure Data Lake Storage Gen2** – Raw & curated data storage
- **Azure Databricks** – Data transformation using PySpark
- **Azure Synapse Analytics** – Data modeling & querying
- **Power BI** – Interactive dashboards
- **Service Principal Authentication**

---

## 📂 Project Layers

### **1. Bronze Layer – Raw Ingestion**
- Raw CSV files ingested from **AdventureWorks dataset** into Azure Data Lake Gen2.
- No transformation applied — pure storage for traceability.

---

### **2. Silver Layer – Transformation in Databricks**
**Key Steps from `silver_layer_refer.ipynb`:**
```python
from pyspark.sql.functions import *
from pyspark.sql.types import *

# Load raw CSV from Bronze Layer
df_cal = spark.read.format('csv').option("header",True).option("inferSchema",True)\
    .load('abfss://bronze@awstoragedatalake.dfs.core.windows.net/AdventureWorks_Calendar')

df_cus = spark.read.format('csv').option("header",True).option("inferSchema",True)\
    .load('abfss://bronze@awstoragedatalake.dfs.core.windows.net/AdventureWorks_Customers')

# Example Transformation: Cleaning & Standardizing column names
df_cus_clean = df_cus.withColumnRenamed("CustomerID", "customer_id")\
                     .withColumnRenamed("FirstName", "first_name")\
                     .withColumnRenamed("LastName", "last_name")

# Write transformed data to Silver Layer in Parquet
df_cus_clean.write.mode("overwrite").parquet('abfss://silver@awstoragedatalake.dfs.core.windows.net/AdventureWorks_Customers')
```

**Transformation Highlights:**
- Schema inference for all ingested CSVs.
- Column renaming for consistency.
- Null handling & data type corrections.
- Saved in **Parquet format** for optimized querying in Synapse.

---

### **3. Gold Layer – Synapse External Tables & Views**
**Key Steps from `Create Views Gold.sql`:**
```sql
CREATE VIEW gold.customers AS
SELECT * 
FROM OPENROWSET(
    BULK 'https://awstoragedatalake.blob.core.windows.net/silver/AdventureWorks_Customers/',
    FORMAT = 'PARQUET'
) AS data;
```

**Gold Layer Activities:**
- Created **external tables** in Synapse referencing Silver Layer Parquet files.
- Built **views** for:
  - `calendar`
  - `customers`
  - `products`
  - `returns`
  - `sales`
  - `subcat`
  - `territories`

---

## 📊 Power BI Dashboard
- Connected Power BI to Synapse Gold Layer.
- Built reports showing:
  - Sales trends by territory.
  - Product performance analysis.
  - Return rate analytics.

---

## 🎯 Key Learnings
- Designing cloud-based data pipelines with Azure services.
- Implementing the Medallion Architecture.
- Optimizing big data queries using Parquet & Synapse external tables.
- Automating data refresh with ADF scheduling.
- Creating interactive analytics in Power BI.

---
