# E-commerce Sales Analytics Pipeline

This repository contains the code, configuration, and documentation for a daily batch data pipeline that processes e-commerce transaction data. The pipeline ingests raw data from AWS S3, applies a series of cleaning and transformation steps, and loads it into a structured Delta Lake warehouse within Databricks Unity Catalog. The final output provides reliable, aggregated sales metrics for business intelligence, analytics, and reporting.

## Table of Contents

  - [Overview](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#overview)
  - [Architecture Diagram](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#architecture-diagram)
  - [Medallion Architecture](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#medallion-architecture)
      - [Bronze Layer: Raw Data Ingestion](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#bronze-layer-raw-data-ingestion)
      - [Silver Layer: Cleaned & Conformed Data](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#silver-layer-cleaned--conformed-data)
      - [Gold Layer: Aggregated Business Metrics](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#gold-layer-aggregated-business-metrics)
  - [Data Source](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#data-source)
  - [Technology Stack](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#technology-stack)
  - [Project Structure](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#project-structure)
  - [Setup and Deployment](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#setup-and-deployment)
  - [Orchestration and Scheduling](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#orchestration-and-scheduling)
  - [Data Quality, Error Handling, and Alerting](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#data-quality-error-handling-and-alerting)
  - [How to Contribute](https://github.com/Kathulahareesh/E-COMMERCE_SALES_ANALYTICS_PIPELINE/tree/main#how-to-contribute)

## Overview

The primary goal of this pipeline is to create a single source of truth for sales analytics. It automates the process of transforming raw, partitioned CSV files into actionable, high-quality datasets. This enables data analysts and business stakeholders to easily query sales performance, track key metrics like customer lifetime value (CLV), and build insightful dashboards.

**Key Features:**

  * **Automated Daily Ingestion:** Processes new transaction data every day.
  * **Medallion Architecture:** Structures data into Bronze, Silver, and Gold layers for clarity and reliability.
  * **Data Quality Assurance:** Implements checks to ensure data accuracy and consistency.
  * **Robust Error Handling:** Captures and logs errors without failing the entire pipeline.
  * **Scalability:** Built on Databricks and Delta Lake to handle growing data volumes.
  * **Version Control:** Integrated with Git for CI/CD and collaborative development.

## Architecture Diagram

The pipeline follows a standard ETL (Extract, Transform, Load) pattern using a multi-hop Medallion Architecture within the Databricks Lakehouse Platform.

```
+----------------+      +--------------------------+      +-------------------------+      +------------------------+      +-------------------+
|   AWS S3       |      |      Bronze Layer        |      |      Silver Layer       |      |      Gold Layer        |      |   BI & Analytics  |
| (Raw CSVs)     |----->| (Raw Delta Table)        |----->| (Cleaned Delta Table)   |----->| (Aggregated Views)     |----->| (Dashboards, SQL) |
| `.../YYYY/MM/DD/`|      | `order_transactions`     |      | `orders_cleaned`        |      | `sales_metrics`        |      |                   |
+----------------+      +--------------------------+      +-------------------------+      +------------------------+      +-------------------+
```

## Medallion Architecture

The data is processed through three distinct layers, each serving a specific purpose. All tables are managed by **Unity Catalog**.

### Bronze Layer: Raw Data Ingestion

The Bronze layer contains the raw, unaltered data ingested directly from the source. It serves as a historical archive and a source for reprocessing if needed.

  * **Catalog:** `ecommerce_catalog`
  * **Schema:** `raw`
  * **Table:** `order_transactions`
  * **Description:** This table is a direct, one-to-one copy of the source CSV files for a given day. Data is appended with metadata columns like `ingestion_date` and `source_file_name`.

### Silver Layer: Cleaned & Conformed Data

The Silver layer provides a validated, enriched, and queryable view of the data. This is where data quality rules, cleaning, and standardization are applied.

  * **Catalog:** `ecommerce_catalog`
  * **Schema:** `processed`
  * **Table:** `orders_cleaned`
  * **Transformations:**
      * **Schema Enforcement:** Applies a consistent schema to all incoming data.
      * **Data Type Casting:** Converts strings to appropriate types (e.g., `timestamp`, `decimal`, `integer`).
      * **Deduplication:** Removes duplicate order records based on `order_id`.
      * **Standardization:** Formats date fields to `YYYY-MM-DD HH:MM:SS` and standardizes currency codes.
      * **Enrichment:** Joins with dimension tables (e.g., product categories, customer details) to add context.
      * **Null Value Handling:** Imputes or flags nulls in critical columns.

### Gold Layer: Aggregated Business Metrics

The Gold layer contains business-level aggregations and KPIs optimized for analytics and reporting. These tables are often dimensionally modeled and power BI dashboards.

  * **Catalog:** `ecommerce_catalog`
  * **Schema:** `analytics`
  * **Table:** `sales_metrics`
  * **Description:** This table provides pre-calculated, aggregated metrics to accelerate reporting.
  * **Aggregations:**
      * Daily total revenue and order count.
      * Revenue per product category.
      * Customer Lifetime Value (CLV) calculations.
      * New vs. returning customer counts.
      * Average order value (AOV).

## Data Source

  * **Source System:** AWS S3
  * **Dataset:** [Kaggle "E-commerce Dataset"](https://www.kaggle.com/datasets/carrie1/ecommerce-data) (or a similar structure)
  * **S3 Path:** `s3://bucket/raw/ecommerce_sales/YYYY/MM/DD/`
  * **Format:** CSV
  * **Partitioning:** Data is expected to be partitioned by year, month, and day in the S3 bucket.

## Technology Stack

  * **Cloud Provider:** AWS
  * **Data Storage:** AWS S3, Delta Lake
  * **Data Platform:** Databricks
  * **Data Governance:** Unity Catalog
  * **Orchestration:** Databricks Workflows
  * **Core Libraries:** PySpark, Pandas, Delta-Spark
  * **Version Control:** Git / GitHub / GitLab

## Project Structure

```
.
├── notebooks/
│   ├── 01-Bronze-Ingestion.py
│   ├── 02-Silver-Transformation.py
│   └── 03-Gold-Aggregation.py
├── src/
│   ├── transformations.py
│   └── utils.py
├── tests/
│   ├── test_transformations.py
│   └── conftest.py
├── conf/
│   ├── pipeline_config.yml
│   └── schemas.json
├── .gitignore
└── README.md
```

  * **notebooks/:** Contains the Databricks notebooks that define the ETL steps for each layer.
  * **src/:** Reusable Python modules for transformations, utilities, and helper functions.
  * **tests/:** Unit tests for the functions defined in `src/`.
  * **conf/:** Configuration files for table names, schemas, paths, and environment-specific settings.

## Setup and Deployment

1.  **Clone Repository:** Clone this repository into your local machine.
2.  **Databricks Repos:** Configure a Databricks Repo to point to this Git repository. This will sync the code to your Databricks workspace.
3.  **Secrets & Configuration:**
      * Set up a Databricks secret scope to securely store AWS credentials (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`).
      * Update the configuration file `conf/pipeline_config.yml` with your specific S3 bucket names, Unity Catalog paths, and other parameters.
4.  **Create Databricks Job:**
      * Navigate to Databricks Workflows and create a new job.
      * Create three sequential tasks, each pointing to a notebook in the `notebooks/` directory (`01-Bronze`, `02-Silver`, `03-Gold`).
      * Configure the job to run on a suitable cluster.
      * Set the schedule for the job.

## Orchestration and Scheduling

  * **Orchestrator:** Databricks Workflows
  * **Schedule:** The pipeline is configured to run as a daily batch job.
  * **Trigger Time:** **3:00 AM UTC**, daily.
  * **Backfilling:** The notebooks are parameterized by date, allowing for easy backfilling of historical data by running the job with a specific date parameter.

## Data Quality, Error Handling, and Alerting

  * **Error Logging:** All exceptions and processing errors are caught and logged to the `ecommerce_catalog.logs.etl_errors` table. Each log entry includes the error message, timestamp, notebook name, and step where the failure occurred.
  * **Data Quality Checks:**
      * **Schema Validation:** Ensures incoming data conforms to the expected schema.
      * **Expectations:** Uses libraries like `pyspark.sql.functions` or external frameworks to validate data (e.g., `order_id` is not null, `order_total` is positive).
      * Anomalies identified during these checks are flagged and can trigger alerts.
  * **Alerting:**
      * **Job Failure:** Databricks Workflows is configured to send email notifications to a specified distribution list on job failure.
      * **Data Anomalies:** If a critical data quality check fails, a custom alert is sent via email, providing details about the anomaly.

## How to Contribute

We welcome contributions\! Please follow these steps:

1.  Fork the repository.
2.  Create a new feature branch (`git checkout -b feature/your-feature-name`).
3.  Make your changes and add unit tests where applicable.
4.  Ensure all tests pass.
5.  Submit a Pull Request with a clear description of your changes.

-----
