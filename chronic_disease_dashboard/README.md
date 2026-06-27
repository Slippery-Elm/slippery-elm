# Project 1: Chronic Disease Population Health Dashboard (Birmingham Risk Tracking)

**Repository Track:** Regional Epidemiological Informatics  
**Data Infrastructure:** Python ETL ➔ Google BigQuery Cloud Warehouse ➔ SQL Aggregation Engine ➔ Tableau Public Presentation  
**Active Production Stage:** Phase 3 - Dashboard Implementation & Executive Insights Completed  

---

## 🔍 Data Source Architecture & Variable Mapping Blueprint
Because raw epidemiological data relies on optimized binary numeric storage codes, the database table uses specific variable keys extracted directly from the CDC's 2017 BRFSS structural definitions.

### 📊 Master Schema Target Definition
The pipeline transforms raw source metadata into the following clean SQL relational tracking structure:

| Source Code Variable | Relational Column Target | Data Type Constraints | Epidemiological Health Matrix Metric |
| :--- | :--- | :--- | :--- |
| `SEQNO` | `respondent_id` | `VARCHAR(20) PRIMARY KEY` | Unique cryptographic survey participant key token identifier. |
| `_STATE` | `state_numeric_code` | `INT NOT NULL` | Standardized Federal Information Processing Series (FIPS) Key. |
| `_MICHD` | `has_cardiovascular_disease` | `INT` | Calculated Status Check: Coronary Heart Disease or Myocardial Infarction. |
| `_TOTINDA` | `exercises_past_30_days` | `INT` | Calculated Lifestyle Metric: Leisure-Time Physical Activity Engagement. |
| `_FRUTSUM` | `daily_fruit_intake_g` | `NUMERIC(6,2)` | Continuous Scale: Computed total daily fruit volume consumption in metric grams. |

---

## 🛠️ Data Warehouse Pipeline Milestones
* [x] Initialize dedicated GitHub production directory.
* [x] Extract binary metadata dictionary parameters from local `.sav` file.
* [x] Generate programmatic SQL DDL (`CREATE TABLE`) schema definition script.
* [x] Execute Extract-Transform-Load (ETL) file conversion routines via Python pipeline.
* [x] Run analytical DQL reporting queries to calculate local risk cluster densities.
* [x] Port analytical data outputs into Tableau Public for interactive presentation.

---

## 🐍 Phase 1: File Ingestion & Data Transformation (Python/Pandas)
To bypass local system memory constraints and convert the raw SPSS binary schema into an open-source, cloud-ready interchange format, a programmatic Python ETL script was executed via Google Colab.

```python
import pandas as pd
import pyreadstat

# 1. Ingest raw binary SPSS file structures 
# (Locally mounted or warehouse-staged data stream)
binary_source_path = "BRFSS_2017_Birmingham_subset.sav"
df, meta = pyreadstat.read_sav(binary_source_path)

# 2. Standardize schema headers to absolute uppercase to ensure mapping sync
df.columns = df.columns.str.upper()

# 3. Isolate localized target clinical observations 
source_columns = ['_MICHD', '_TOTINDA', '_FRUTSU1']
df_filtered = df[source_columns].copy()

# 4. Apply structural schema dictionary definitions to target metrics
schema_mapping = {
    '_MICHD': 'has_cardiovascular_disease',
    '_TOTINDA': 'exercises_past_30_days',
    '_FRUTSU1': 'daily_fruit_intake_g'
}
df_filtered.rename(columns=schema_mapping, inplace=True)

# 5. Programmatically inject administrative keys and structural sequence IDs
# Generates a unique index string to substitute for omitted baseline survey key tokens
df_filtered['respondent_id'] = [f"BHM_2017_{i+1:05d}" for i in range(len(df_filtered))]
df_filtered['state_numeric_code'] = 1  # Hardcoded Alabama FIPS alignment tag

# 6. Restructure column ordering to align with Cloud Data Warehouse DDL schemas
final_column_order = [
    'respondent_id', 
    'state_numeric_code', 
    'has_cardiovascular_disease', 
    'exercises_past_30_days', 
    'daily_fruit_intake_g'
]
df_production = df_filtered[final_column_order].copy()

# 7. Execute analytical cleaning constraints
df_production.dropna(subset=['has_cardiovascular_disease', 'exercises_past_30_days'], inplace=True)
df_production['has_cardiovascular_disease'] = df_production['has_cardiovascular_disease'].astype(int)
df_production['exercises_past_30_days'] = df_production['exercises_past_30_days'].astype(int)

# 8. Export clean, unindexed flat file for Google BigQuery Cloud Ingestion
df_production.to_csv("birmingham_brfss_clean.csv", index=False)
print(f"ETL Extraction Complete. Processed {len(df_production):,} clean public health records.")

```
---

## ☁️ Phase 2: Cloud Data Warehousing & Aggregate Metrics (Google BigQuery SQL)
The structured dataset was transferred into a Google BigQuery cloud instance. To prepare targeted, optimized matrices for presentation layouts, an analytical Data Query Language (DQL) script executed multidimensional aggregations to segment clinical outcomes against behavioral variables.

``` SQL
SELECT 
    exercises_past_30_days,
    has_cardiovascular_disease,
    COUNT(*) AS patient_volume,
    ROUND(AVG(daily_fruit_intake_g), 2) AS avg_fruit_grams
FROM 
    `birmingham_cloud_warehouse.epidemiology_tables.birmingham_clean_epidemiology`
GROUP BY 
    exercises_past_30_days,
    has_cardiovascular_disease
ORDER BY 
    patient_volume DESC;

```
---

## 📊 Phase 3: Interactive Executive Dashboard (Tableau Public)
Metadata Translation Keys Applied: 
* Exercises Past 30 Days: 1 = Active | 2 = Sedentary | 9 = Unknown
* Has Cardiovascular Disease: 1 = Has Heart Disease | 2 = No Heart Disease

### [** Click here to view the live interactive dashboard on Tableau Public **](https://public.tableau.com/app/profile/rachel.asante/viz/Birmingham_Chronic_Disease_Population_Dashboard/Dashboard1?publish=yes)

---

## 🧠 Phase 4: Executive Clinical Brief & Intervention Strategy

### 📊 Epidemic Insight & Risk Identification
In the absence of secondary preventive measures like regular physical activity and optimal nutrition, the likelihood of functional degradation increases. Within the Birmingham metro cohort, these compounding behavioral comorbidities are starkly visible: sedentary individuals with pre-existing cardiovascular conditions recorded a daily fruit intake of only 90.18 grams, compared to 111.05 grams among their non-chronic counterparts.

### 🏥 Institutional Capacity Demands
In 2017, a baseline cohort of 340 sedentary individuals without heart disease and 54 sedentary individuals with active heart disease relied on the healthcare system for general care. These volumetric surges led to high resource expenditures, increasing fiscal volatility for the hospital system. Therefore, it is imperative to implement targeted community-based interventions to support sedentary cohorts, thereby mitigating avoidable inpatient admissions.

### 🎯 Actionable Public Health Recommendations
To address the critical nutritional deficits identified within high-risk cohorts, health administrators must expand community-based nutritional distribution networks. Specifically, establishing subsidized nutritional incentive programs—such as mobile farmers markets deployed directly within underserved neighborhoods—can mitigate structural food insecurity barriers. Integrating clinical referral pipelines with these localized agricultural networks will empower high-risk individuals to optimize modifiable behaviors and sustain long-term secondary prevention.
