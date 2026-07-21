# HDB Data ETL Pipeline

An automated, modular ETL (Extract, Transform, Load) pipeline implemented in Databricks for processing HDB (Housing & Development Board) datasets. The workflow Orchestrates child notebooks to handle data ingestion, cleaning, validation, and transformation.

---

## 📌 Architecture Overview

The master notebook orchestrates sequential steps using `dbutils.notebook.run()`. Each step receives file paths and configuration parameters dynamically via Databricks widgets.

+---------------------+
|  Input CSV Files    |
+----------+----------+
|
v
+----------+----------+
|  1. Data Reading    | ---> Raw / Master Data CSV
+----------+----------+
|
v
+----------+----------+
|  2. Data Cleaning   | ---> Cleaned Data CSV & Failed Records CSV
+----------+----------+
|
v
+----------+----------+
|  3. Data Validation | ---> Passed Validation
+----------+----------+
|
v
+----------+----------+
| 4. Data Transform   | ---> Transformed Data CSV & Hashed Data CSV
+---------------------+
