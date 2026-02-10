LA Crime Analysis Pipeline
Overview
An end-to-end ETL pipeline for analyzing Los Angeles crime data using Databricks, Delta Live Tables, and Tableau. The pipeline processes ~1 million crime incident records from LA City's Open Data API, implements medallion architecture (Bronze → Silver → Gold), and delivers interactive dashboards for crime trend analysis.
Architecture
┌─────────────────────────────────────────────────────────────────────┐
│                        LA City Open Data API                         │
│                    (~1M crime incident records)                      │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
                    ┌────────────────┐
                    │  Bronze Layer  │
                    │   Raw Data     │
                    │  (1,004,991    │
                    │   records)     │
                    └────────┬───────┘
                             │
                             ▼
                    ┌────────────────┐
                    │  Silver Layer  │
                    │  Cleaned &     │
                    │  Validated     │
                    │  (100% retain) │
                    └────────┬───────┘
                             │
                             ▼
                    ┌────────────────┐
                    │   Gold Layer   │
                    │  Dimensional   │
                    │  Star Schema   │
                    └────────┬───────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
  ┌──────────┐        ┌──────────┐        ┌──────────┐
  │ Fact     │        │ 7 Dim    │        │ Tableau  │
  │ Table    │        │ Tables   │        │ Dashboards│
  └──────────┘        └──────────┘        └──────────┘

  Key Features

Medallion Architecture: Bronze (raw) → Silver (cleaned) → Gold (dimensional model)
Data Quality Checks: Date validation, geographic boundary validation, NULL handling, deduplication
SCD Type 2: Implemented for location dimension to track historical changes
Star Schema: 1 fact table + 7 dimension tables following Kimball methodology
Interactive Dashboards: Tableau visualizations for crime trends, geographic analysis, and demographics

Data Pipeline
Bronze Layer

Ingests raw CSV data from LA City Open Data API
Stores in Unity Catalog Volume
1,004,991 records, 28 columns
No transformations applied

Silver Layer

100% record retention (no data loss)
Data quality validations:

Date format validation (YYYYMMDD)
Time range validation (0000-2359)
Geographic boundary checks (LA coordinates)
Victim age validation (1-120 years)


Standardized values for unknown/missing data
Geographic imputation using area centroids

Gold Layer
Fact Table: fact_crime

Grain: One row per crime incident (DR_NO)
7 dimension foreign keys
2 degenerate dimensions
1 measure (is_arrest_flag)

Dimension Tables:
DimensionPurposeKey Attributesdim_dateTime-series analysisyear, quarter, month, day_of_weekdim_timeTime-of-day patternstime_period (Morning/Afternoon/Evening/Night)dim_locationGeographic analysis (SCD Type 2)area, area_name, lat, lon, is_currentdim_crime_typeCrime classificationcrm_cd, crm_cd_desc, part_1_2dim_weaponWeapon trackingweapon_used_cd, weapon_descdim_victimDemographicsage_group, sexdim_statusCase statusstatus_code, status_desc
Data Quality Rules
ValidationRuleActionYear Range1990-2030Invalid → NULLDate FormatYYYYMMDD (8 digits)Invalid → NULLTime Range0000-2359Invalid → NULLLatitude33.7°N - 34.8°NOut of range flaggedLongitude-118.7°W - -118.0°WOut of range flaggedVictim Age1-120 yearsOut of range → NULLCoordinates(0, 0) detectionFlagged as missing
Tech Stack

Processing: Databricks, Delta Live Tables, PySpark
Storage: Unity Catalog, Delta Lake
Visualization: Tableau
Data Modeling: Kimball methodology, Star Schema

Dashboard Insights
The Tableau dashboards answer key business questions:
Crime Trends

Overall crime rate trends over the years
Monthly and quarterly patterns

Time Analysis

Day of week vs. crime correlation
Crime types by time of day

Geographic Analysis

High-crime areas in Los Angeles
Crime hotspot identification

Demographics

Age patterns in crime
Gender-related patterns

Arrests

Juvenile vs. Adult arrest percentages

Setup & Installation
Prerequisites

Databricks workspace with Unity Catalog
Tableau Desktop or Tableau Server
Access to the LA City Open Data API

Steps

Clone Repository

```

cd la-crime-pipeline
```
Configure Databricks
```
pythoncatalog = "workspace"
schema = "crime"
volume = "datastore"


3. **Run Pipeline**
- Execute the Bronze layer notebook to ingest data
- Execute Silver layer notebook for cleaning
- Execute the Gold layer notebook for the dimensional model

4. **Connect Tableau**
- Connect Tableau to Databricks
- Import dimension and fact tables
- Configure relationships
```
Results
MetricValueTotal Records Processed1,004,991Data Retention Rate100%Dimension Tables7Fact Table Records~1MDashboard Views6
