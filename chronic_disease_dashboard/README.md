# Project 1: Chronic Disease Population Health Dashboard (Birmingham Risk Tracking)

**Repository Track:** Regional Epidemiological Informatics  
**Data Infrastructure:** SQL Engine Extraction Engine -> Tableau Visualization Pipeline  
**Active Production Stage:** Phase 2 - Metadata Dictionary Resolution  

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
* [ ] Extract binary metadata dictionary parameters from local `.sav` file.
* [ ] Generate programmatic SQL DDL (`CREATE TABLE`) schema definition script.
* [ ] Execute Extract-Transform-Load (ETL) file conversion routines via Python pipeline.
* [ ] Run analytical DQL reporting queries to calculate local risk cluster densities.
* [ ] Port analytical data outputs into Tableau Public for geospatial risk visualization mapping.
