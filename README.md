# DataCo Supply Chain Analytics Pipeline

A comprehensive supply chain analytics solution implementing the medallion architecture (Bronze-Silver-Gold) to transform raw supply chain data into actionable KPIs for inventory management and delivery performance.

## Overview

This project processes the DataCo supply chain dataset (180,519 orders across 53 attributes) through three progressive layers, each adding value through data cleaning, enrichment, and business logic.

## Architecture

### Bronze Layer: Raw Data Ingestion
**Notebook:** `P2_01_Bronze_DataCo_Ingestion`

**Purpose:** Initial data ingestion and standardization

**Key Operations:**
* Loads raw supply chain data from `workspace.default.data_co_supply_chain_dataset`
* Standardizes column names (lowercase, snake_case, remove special characters)
* Converts date strings to proper timestamp format
* Removes sensitive data (emails, passwords) and unused fields (product images, descriptions)
* Output: `workspace.default.bronze_dataco` (180,519 rows × 49 columns)

### Silver Layer: Data Cleaning & Enrichment
**Notebook:** `P2_02_Silver_DataCo_Cleaning`

**Purpose:** Data quality improvement and feature engineering

**Key Operations:**
* Null value analysis and treatment
  * Fills missing customer last names with "Unknown"
  * Fills missing zipcodes with 0
  * Drops `order_zipcode` (86% null values)
* Adds derived analytical columns:
  * `is_late`: Binary flag for late deliveries
  * `shipping_delay_days`: Difference between actual and scheduled shipping time
  * `order_year`, `order_month`, `order_yearmonth`: Temporal dimensions for time-series analysis
* Output: `workspace.default.silver_dataco` (180,519 rows × 53 columns)

### Gold Layer: Business KPIs
**Notebook:** `P2_03_Gold_KPI_Pipeline`

**Purpose:** Calculate business-critical supply chain metrics

**KPIs Implemented:**

#### 1. Fill Rate (On-Time Delivery Rate)
* **Formula:** (1 - Late Orders / Total Orders) × 100
* **Dimensions:** Year, Month, Region
* **Table:** `workspace.default.gold_kpi_fill_rate`

#### 2. Inventory Turnover & Days of Inventory (DOI)
* **Turnover Formula:** Total Sales / Avg Inventory Value
* **DOI Formula:** 365 / Inventory Turnover
* **Dimensions:** Year, Month, Category
* **Table:** `workspace.default.gold_kpi_inventory`

#### 3. Slow Movers
* **Logic:** Products with order volume < 50% of category average
* **Dimensions:** Category, Product
* **Table:** `workspace.default.gold_kpi_slow_movers`

#### 4. Excess & Obsolete Inventory (E&O)
* **Logic:** High-price products with below-average order volume
* **Risk Flag:** Price > category avg AND orders < category avg
* **Table:** `workspace.default.gold_kpi_eo`

#### 5. Forecast vs Real Consumption
* **Forecast Method:** 3-month moving average
* **Accuracy:** (1 - |Real - Forecast| / Real) × 100
* **Variance:** (Real - Forecast) / Forecast × 100
* **Table:** `workspace.default.gold_kpi_forecast`

## Data Flow

```
Source Data (data_co_supply_chain_dataset)
         ↓
  [Bronze Layer]
   Data Ingestion & Standardization
   (bronze_dataco)
         ↓
  [Silver Layer]
   Cleaning & Feature Engineering
   (silver_dataco)
         ↓
  [Gold Layer]
   Business KPI Calculation
         ├── gold_kpi_fill_rate
         ├── gold_kpi_inventory
         ├── gold_kpi_slow_movers
         ├── gold_kpi_eo
         └── gold_kpi_forecast
```

## Tech Stack

* **Platform:** Databricks
* **Language:** Python (PySpark)
* **Storage Format:** Delta Lake
* **Catalog:** Unity Catalog (`workspace.default.*`)

## Execution Order

1. Run `P2_01_Bronze_DataCo_Ingestion` to create the bronze layer
2. Run `P2_02_Silver_DataCo_Cleaning` to create the silver layer
3. Run `P2_03_Gold_KPI_Pipeline` to generate all KPI tables

## Output Tables

| Layer  | Table Name | Rows | Purpose |
|--------|-----------|------|--------|
| Bronze | `workspace.default.bronze_dataco` | 180,519 | Standardized raw data |
| Silver | `workspace.default.silver_dataco` | 180,519 | Cleaned & enriched data |
| Gold | `workspace.default.gold_kpi_fill_rate` | Aggregated | On-time delivery metrics |
| Gold | `workspace.default.gold_kpi_inventory` | Aggregated | Inventory turnover & DOI |
| Gold | `workspace.default.gold_kpi_slow_movers` | Product-level | Low-demand product analysis |
| Gold | `workspace.default.gold_kpi_eo` | Product-level | Excess inventory risk |
| Gold | `workspace.default.gold_kpi_forecast` | Time-series | Forecast accuracy tracking |

## Use Cases

* **Supply Chain Operations:** Monitor delivery performance and identify bottlenecks
* **Inventory Management:** Optimize stock levels and identify slow-moving inventory
* **Demand Planning:** Track forecast accuracy and adjust planning models
* **Financial Analysis:** Understand inventory carrying costs and turnover efficiency
* **Strategic Planning:** Regional performance comparison and resource allocation

## License

See [LICENSE](LICENSE) file for details.