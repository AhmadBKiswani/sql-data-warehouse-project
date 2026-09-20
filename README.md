# 🗄️ SQL Data Warehouse: Medallion Architecture

[![GitHub Repository](https://img.shields.io/badge/GitHub-Repository-181717?logo=github)](https://github.com/AhmadBKiswani/sql-data-warehouse-project)
[![SQL Server](https://img.shields.io/badge/Microsoft%20SQL%20Server-CC2927?logo=microsoft-sql-server&logoColor=white)](#)
[![T-SQL](https://img.shields.io/badge/T--SQL-004683?logo=microsoft-sql-server&logoColor=white)](#)

This repository contains an end-to-end robust Data Warehouse solution built entirely with **Microsoft SQL Server** and **T-SQL**. The project implements a modern **Medallion Architecture** (Bronze, Silver, and Gold layers) to extract, cleanse, integrate, and transform raw data from disparate Customer Relationship Management (CRM) and Enterprise Resource Planning (ERP) systems into a business-ready Star Schema optimized for analytics and BI reporting.

## 📌 Project Overview

The primary objective of this project is to showcase advanced database management, ETL (Extract, Transform, Load) design, and dimensional data modeling. The pipeline processes raw `.csv` files, applies rigorous data quality rules, and integrates multiple source tables to establish a single source of truth for sales and customer analytics. 

Key technical implementations include:
* **Stored Procedures:** Automated batch processing for data ingestion and transformation.
* **Error Handling:** Robust `TRY...CATCH` blocks to log and manage pipeline failures.
* **Performance Tracking:** Execution time calculations for monitoring batch load durations.
* **Data Cleansing:** Handling missing values, standardizing formats, and enforcing mathematical consistency.
* **Dimensional Modeling:** Constructing a Star Schema with Fact and Dimension tables using surrogate keys.

---

## 🏗️ Architecture & Data Flow

The data pipeline follows a strict three-tier Medallion Architecture, ensuring progressive data refinement.

### 🥉 1. Bronze Layer (Raw Data Ingestion)
The landing zone for raw operational data.
* **Objective:** Quickly load raw CSV files into the database without applying any transformations.
* **Process:** Uses `TRUNCATE` and `BULK INSERT` T-SQL commands within a stored procedure (`bronze.load_bronze`) to fully refresh tables on each run.
* **Sources:**
  * **CRM:** `crm_cust_info`, `crm_prd_info`, `crm_sales_details`
  * **ERP:** `erp_cust_az12`, `erp_loc_a101`, `erp_px_cat_g1v2`

### 🥈 2. Silver Layer (Cleansing & Standardization)
The conformed and integrated layer where data quality is established.
* **Objective:** Clean, standardize, and integrate data from the Bronze layer.
* **Process:** Executes the `silver.load_silver` stored procedure to perform the following:
  * **Deduplication:** Uses window functions (`ROW_NUMBER()`) to identify and keep only the latest records.
  * **Standardization:** Normalizes categorical values (e.g., mapping 'M'/'S' to 'Married'/'Single', 'F'/'M' to 'Female'/'Male').
  * **Data Integrity:** Cleanses text strings (e.g., removing 'NAS' prefixes from IDs), converts integer date formats to standard `DATE` types, and nullifies impossible future dates.
  * **Derived Logic:** Cross-checks and recalculates missing or invalid financial metrics (`sls_sales = sls_quantity * sls_price`).
  * **Auditing:** Appends a `dwh_create_date` timestamp to track when records were processed.

### 🥇 3. Gold Layer (Business & Presentation)
The semantic layer optimized for end-user reporting and BI dashboards (e.g., Power BI).
* **Objective:** Create a denormalized Star Schema consisting of analytics-ready views.
* **Process:** Connects the cleaned CRM and ERP data using `LEFT JOIN`s and handles missing dimension attributes using `COALESCE`.
* **Data Model (Star Schema):**
  * **`dim_customers`:** Integrated dimension combining customer profiles, demographic details (birthdates), and geographical locations. Generates a unique surrogate `customer_key`.
  * **`dim_products`:** Dimension detailing product categories, lines, and lifecycle dates.
  * **`fact_sales`:** The core transactional table containing standardized sales amounts, quantities, and prices, linked to the dimensions via foreign keys.

---

## 📂 Repository Structure

The repository is organized into SQL scripts representing the DDL (Data Definition Language) and DML (Data Manipulation Language) for each architectural layer, alongside the source and output data files.

* **Architecture Diagrams:** Contains the visual architecture layout, Data Flow Diagram, Integration Model, and Data Model mappings.
* **Bronze Layer:** `DDL Bronze.sql`, `DML Bronze.sql`
* **Silver Layer:** `ddl_silver.sql`, `proc_load_silver.sql`, and individual transformation scripts.
* **Gold Layer:** `Dimension Customer.sql`, `Dimension Products.sql`, `Fact Sales.sql`
* **Datasets:** Contains raw source `.csv` files and the finalized Gold data mart exports.

---

## 🚀 How to Run the Project

**1. Clone the Repository**
<br>

```bash
git clone [https://github.com/AhmadBKiswani/sql-data-warehouse-project.git](https://github.com/AhmadBKiswani/sql-data-warehouse-project.git)
```

**2. Setup Database & Schemas**
<br>Open SQL Server Management Studio (SSMS) or Azure Data Studio. Create a new database and the required Medallion schemas:

```sql
CREATE DATABASE DataWarehouse;
GO
USE DataWarehouse;
GO
CREATE SCHEMA bronze;
GO
CREATE SCHEMA silver;
GO
CREATE SCHEMA gold;
GO
```

**3. Build the Table Structures (DDL)**
<br>Execute the table creation scripts to build the foundation for the raw and cleansed data layers:
* Run `DDL Bronze.sql` to create the bronze tables.
* Run `ddl_silver.sql` to create the silver tables.

**4. Update File Paths for Data Ingestion**
<br>SQL Server requires absolute local file paths to execute a `BULK INSERT`. 
* Open `DML Bronze.sql`.
* Find the `FROM` clauses (e.g., `'C:\Users\User\Desktop\sql-data-warehouse-project\datasets\source_crm\cust_info.csv'`).
* Replace the paths with the exact local path where you cloned this repository.

**5. Compile the Stored Procedures**
<br>Execute the stored procedure scripts to compile them into the database. This does *not* run the data load; it simply saves the logic.
* Run `DML Bronze.sql` to compile `bronze.load_bronze`.
* Run `proc_load_silver.sql` to compile `silver.load_silver`.

**6. Execute the ETL Pipeline**
<br>Run the following commands in a new query window to extract the CSV data, load it into the Bronze layer, and transform it into the Silver layer.

```sql
EXEC bronze.load_bronze;
EXEC silver.load_silver;
```

*(Note: Check the messages tab to view the batch execution times and ensure there were no errors).*

**7. Generate the Gold Data Mart**
<br>Execute the view creation scripts to build the Star Schema for reporting:
* Run `Dimension Customer.sql`
* Run `Dimension Products.sql`
* Run `Fact Sales.sql`

**8. Connect & Analyze**
<br>The Gold layer views are now fully populated and ready for analysis. Connect Power BI directly to the `DataWarehouse` database and import the `gold.dim_customers`, `gold.dim_products`, and `gold.fact_sales` views to begin building dashboards.

---

## 👨‍💻 Author
**Ahmad Kiswani**
* [GitHub](https://github.com/AhmadBKiswani)
* [LinkedIn](https://www.linkedin.com/in/ahmadbkiswani)
