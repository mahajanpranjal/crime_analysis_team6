# LA Crime Analysis Pipeline

## Overview

An end-to-end ETL pipeline for analyzing **Los Angeles crime data** using **Databricks**, **Delta Live Tables**, and **Tableau**. The pipeline processes **~1 million crime incident records** from LA City's Open Data API, implements **medallion architecture** (Bronze → Silver → Gold), and delivers interactive dashboards for crime trend analysis.

## Architecture

**Data Flow:**
```
LA City Open Data API (~1M records)
         ↓
   Bronze Layer (Raw Data - 1,004,991 records)
         ↓
   Silver Layer (Cleaned & Validated - 100% retention)
         ↓
   Gold Layer (Star Schema - 1 Fact + 7 Dimensions)
         ↓
   Tableau Dashboards
```

## Key Features

- **Medallion Architecture:** Bronze (raw) → Silver (cleaned) → Gold (dimensional model)
- **Data Quality Checks:** Date validation, geographic boundary validation, NULL handling, deduplication
- **SCD Type 2:** Implemented for the location dimension to track historical changes
- **Star Schema:** 1 fact table + 7 dimension tables following Kimball methodology
- **Interactive Dashboards:** Tableau visualizations for crime trends, geographic analysis, and demographics

## Data Pipeline

### Bronze Layer
- Ingests raw CSV data from **LA City Open Data API**
- Stores in **Unity Catalog Volume**
- **1,004,991 records**, 28 columns
- No transformations applied

### Silver Layer
- **100% record retention** (no data loss)
- Data quality validations:
  - Date format validation (YYYYMMDD)
  - Time range validation (0000-2359)
  - Geographic boundary checks (LA coordinates)
  - Victim age validation (1-120 years)
- Standardized values for unknown/missing data
- Geographic imputation using area centroids

### Gold Layer

**Fact Table:** `fact_crime`
- **Grain:** One row per crime incident (DR_NO)
- 7-dimensional foreign keys
- 2 degenerate dimensions
- 1 measure (is_arrest_flag)

**Dimension Tables:**

| Dimension | Purpose | Key Attributes |
|-----------|---------|----------------|
| **dim_date** | Time-series analysis | year, quarter, month, day_of_week |
| **dim_time** | Time-of-day patterns | time_period (Morning/Afternoon/Evening/Night) |
| **dim_location** | Geographic analysis (SCD Type 2) | area, area_name, lat, lon, is_current |
| **dim_crime_type** | Crime classification | crm_cd, crm_cd_desc, part_1_2 |
| **dim_weapon** | Weapon tracking | weapon_used_cd, weapon_desc |
| **dim_victim** | Demographics | age_group, sex |
| **dim_status** | Case status | status_code, status_desc |

## Data Quality Rules

| Validation | Rule | Action |
|------------|------|--------|
| Year Range | 1990-2030 | Invalid → NULL |
| Date Format | YYYYMMDD (8 digits) | Invalid → NULL |
| Time Range | 0000-2359 | Invalid → NULL |
| Latitude | 33.7°N - 34.8°N | Out of range flagged |
| Longitude | -118.7°W - -118.0°W | Out of range flagged |
| Victim Age | 1-120 years | Out of range → NULL |
| Coordinates | (0, 0) detection | Flagged as missing |

## Tech Stack

- **Processing:** Databricks, Delta Live Tables, PySpark
- **Storage:** Unity Catalog, Delta Lake
- **Visualization:** Tableau
- **Data Modeling:** Kimball methodology, Star Schema

## Dashboard Insights

The Tableau dashboards answer key business questions:

**Crime Trends**
- Overall crime rate trends over the years
- Monthly and quarterly patterns

**Time Analysis**
- Day of week vs. crime correlation
- Crime types by time of day

**Geographic Analysis**
- High-crime areas in Los Angeles
- Crime hotspot identification

**Demographics**
- Age patterns in crime
- Gender-related patterns

**Arrests**
- Juvenile vs. Adult arrest percentages

## Setup & Installation

### Prerequisites
- Databricks workspace with Unity Catalog
- Tableau Desktop or Tableau Server
- Access to LA City Open Data API

### Steps

1. **Clone Repository**
```bash
git clone https://github.com/aishwarya4114/la-crime-pipeline.git
cd la-crime-pipeline
```

2. **Configure Databricks**
```python
catalog = "workspace"
schema = "crime"
volume = "datastore"
```

3. **Run Pipeline**
- Execute the Bronze layer notebook to ingest data
- Execute Silver layer notebook for cleaning
- Execute the Gold layer notebook for the dimensional model

4. **Connect Tableau**
- Connect Tableau to Databricks
- Import dimension and fact tables
- Configure relationships

## Results

| Metric | Value |
|--------|-------|
| **Total Records Processed** | 1,004,991 |
| **Data Retention Rate** | 100% |
| **Dimension Tables** | 7 |
| **Fact Table Records** | ~1M |
| **Dashboard Views** | 6 |

