# HDB Data ETL Pipeline

An automated, modular ETL (Extract, Transform, Load) pipeline implemented in Databricks for processing HDB (Housing & Development Board) datasets. The workflow Orchestrates child notebooks to handle data ingestion, cleaning, validation, and transformation.

---

## 📌 Architecture Overview

The master notebook orchestrates sequential steps using `dbutils.notebook.run()`. Each step receives file paths and configuration parameters dynamically via Databricks widgets.

### **Pipeline Execution Flow**

1. **Input Stage:** Raw CSV files (`/Volumes/workspace/hdb/hdb-dataset/*.csv`)
2. **Step 1 — Data Reading (`./hdb data reading`):**
   * *In:* Input CSV Folder
   * *Out:* Master Data File (`hdb_master_data.csv`)
3. **Step 2 — Data Cleaning (`./hdb data cleaning`):**
   * *In:* Master Data File
   * *Out:* Cleaned Data File (`hdb_cleaned_data.csv`) & Failed Data File (`hdb_failed_data.csv`)
4. **Step 3 — Data Validation (`./hdb data validation`):**
   * *In:* Cleaned Data File
   * *Out:* Validated Dataset
5. **Step 4 — Data Transformation (`./hdb data transformation`):**
   * *In:* Cleaned Data File
   * *Out:* Transformed Data File (`hdb_transformed_data.csv`) & Hashed Data File (`hdb_hashed_data.csv`)

---


---

## 📁 Pipeline Components

The pipeline triggers the following modular child notebooks in order:

1. **Data Reading (`./hdb data reading`)**
   * **Purpose:** Ingests raw CSV files from the input volume and writes/consolidates them into master data storage.
   * **Parameters:** `input_folder`, `master_data_file`

2. **Data Cleaning (`./hdb data cleaning`)**
   * **Purpose:** Handles missing values, standardizes formatting, and isolates bad records for audit purposes.
   * **Parameters:** `master_data_file`, `cleaned_data_file`, `failed_data_file`

3. **Data Validation (`./hdb data validation`)**
   * **Purpose:** Performs schema checks and business logic validations on the cleaned data.
   * **Parameters:** `cleaned_data_file`

4. **Data Transformation (`./hdb data transformation`)**
   * **Purpose:** Applies feature engineering, column transformations, and data hashing/anonymization for downstream consumption.
   * **Parameters:** `cleaned_data_file`, `transformed_data_file`, `hashed_data_file`

---

### **Data Profiling**

**Data Profiling (`./hdb data profiling`)**
   * **Purpose:** Analyzes the master data file to generate descriptive statistics, null value checks, and column profile metrics before cleaning. Detects anomalies in HDB resale flat prices using Grouped Interquartile Range (IQR).
   * *In:* Master Data File
   * *Out:* Data distributions, column summaries, and schema health reports
   * *Parameters:* `master_data_file`


## ⚙️ Configuration Parameters (Widgets)

| Parameter Name | Default Path / Value | Description |
| :--- | :--- | :--- |
| `input_folder` | `/Volumes/workspace/hdb/hdb-dataset/*.csv` | Source directory for raw HDB CSV files |
| `master_data_file` | `/Volumes/workspace/hdb/hdb-output-data/hdb_master_data.csv` | Output path for aggregated master data |
| `cleaned_data_file` | `/Volumes/workspace/hdb/hdb-output-data/hdb_cleaned_data.csv` | Output path for successfully cleaned data |
| `failed_data_file` | `/Volumes/workspace/hdb/hdb-output-data/hdb_failed_data.csv` | Output path for rejected/failed records |
| `transformed_data_file` | `/Volumes/workspace/hdb/hdb-output-data/hdb_transformed_data.csv` | Final output path for transformed dataset |
| `hashed_data_file` | `/Volumes/workspace/hdb/hdb-output-data/hdb_hashed_data.csv` | Output path for hashed/anonymized data |

---

## 🚀 How to Run

1. Open the **`ETL pipeline`** master notebook in Databricks.
2. (Optional) Adjust the input path or output filenames using the widget text fields at the top of the notebook.
3. Click **Run All** or execute the code cell.

---
---

## 💾 Databricks Volume Storage Structure

Data is stored using Unity Catalog Volumes under the `/Volumes/workspace/hdb/` directory path. This provides POSIX-compliant file paths for seamless reading and writing via standard Python tools (`pandas`, `glob`, `os`) and Spark.

### **Directory & File Hierarchy**

```text
/Volumes/workspace/hdb/
├── hdb-dataset/                     <-- INPUT VOLUME (Raw Data Files)
│   ├── ResaleFlatPricesBasedonApprovalDate19901999.csv
│   ├── ResaleFlatPricesBasedonApprovalDate2000Feb2012.csv
│   ├── ResaleFlatPricesBasedonRegistrationDateFromMar2012toDec2014.csv
│   ├── ResaleFlatPricesBasedonRegistrationDateFromJan2015toDec2016.csv
│   └── ResaleFlatPricesBasedonRegistrationDateFromJan2017onwards.csv
│
└── hdb-output-data/                 <-- OUTPUT VOLUME (ETL Artifacts)
    ├── hdb_master_data.csv          # Consolidated raw dataset from all source CSVs
    ├── hdb_cleaned_data.csv         # Cleaned and standardized dataset
    ├── hdb_failed_data.csv          # Isolated rejected/invalid records
    ├── hdb_transformed_data.csv     # Transformed dataset ready for analytics
    └── hdb_hashed_data.csv          # Anonymized/hashed dataset

---

## 🛠️ Infrastructure Setup (Databricks Volumes)

Before running the ETL pipeline, ensure the Unity Catalog volumes are created using the following SQL commands:

```sql
-- Step 1: Ensure catalog and schema exist
CREATE CATALOG IF NOT EXISTS workspace;
CREATE SCHEMA IF NOT EXISTS workspace.hdb;

-- Step 2: Create Volume for raw input dataset CSV files
CREATE VOLUME IF NOT EXISTS workspace.hdb.`hdb-dataset`;

-- Step 3: Create Volume for output datasets and processed artifacts
CREATE VOLUME IF NOT EXISTS workspace.hdb.`hdb-output-data`;



## Notes

* **Exception Handling Note:** `dbutils.notebook.exit()` raises a system exception under the hood in Python. If wrapped inside a generic `try...except Exception` block, catching `Exception` will intercept `exit()`. To prevent success messages from triggering the `except` block, ensure you catch specific exceptions (e.g., `Py4JJavaError`) or handle the exit logic outside the generic `try...except`.
