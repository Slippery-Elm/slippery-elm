# Public Health Data Analytics Portfolio (June 23, 2026)

## Project 1: Chronic Disease Population Health Dashboard
**Track:** Healthcare Analytics  
**Data Source:** Mock BRFSS Chronic Condition Data  

### Objectives
Identify regional wellness zones by isolating patients who are sedentary but currently free of diagnosed cardiovascular disease. This allows healthcare administrators to implement preemptive preventive health interventions.

### Optimized SQL Execution Script
```sql
SELECT
     state_name,
     COUNT(respondent_id) AS sedentary_non_cardiovascular_patient_count
FROM
      brfss_chronic_conditions
WHERE
      has_cardiovascular_disease = 0
      AND exercises_past_30_days = 0
GROUP BY
       state_name
ORDER BY
        sedentary_non_cardiovascular_patient_count DESC;

```

# Public Health SQL Portfolio: Vaccine Doses by Age Group (August 12, 2026)

## Overview

This query answers a common public health reporting question:
**"How many vaccine doses have been administered to each age group, including patients with no vaccination record on file?"**

It combines several core SQL concepts: `LEFT JOIN`, `CASE WHEN`, `GROUP BY`, and `HAVING`.

## Sample Tables

**`patients`**

| patient_id | last_name | zip_code |
|---|---|---|
| 101 | Alvarez | 30093 |
| 102 | Chen | 30071 |
| 103 | Diallo | 30093 |
| 104 | Nguyen | 30045 |
| 106 | Osei | 30022 |

**`immunization_log`**

| patient_id | vaccine | dose_date | age |
|---|---|---|---|
| 101 | MMR | 2024-01-15 | 4 |
| 101 | Flu | 2024-10-02 | 4 |
| 102 | MMR | 2023-11-20 | 6 |
| 103 | Flu | 2024-10-05 | 34 |
| 103 | Hep B | 2024-03-11 | 34 |
| 104 | Flu | 2024-09-28 | 71 |
| 105 | Flu | 2024-10-10 | 68 |

## The Query

```sql
SELECT 
    -- Bucket each patient into an age group.
    -- Patients with no immunization record at all (e.g. patient 106)
    -- will have a NULL age after the LEFT JOIN, so that case is
    -- handled explicitly rather than falling through unexpectedly.
    CASE 
        WHEN age <= 18 THEN 'Pediatric'
        WHEN age BETWEEN 19 AND 64 THEN 'Adult'
        WHEN age IS NULL THEN 'Unknown'
        ELSE 'Senior'
    END AS age_group,

    -- Count how many dose records fall into each age group.
    -- Patients with zero doses will show up here as part of the
    -- group total, but contribute 0 to the COUNT.
    COUNT(*) AS total_doses

FROM patients

-- LEFT JOIN keeps every patient from the patients table, even those
-- with no matching rows in immunization_log (e.g. patient 106, Osei).
-- Unmatched rows return NULL for vaccine, dose_date, and age.
LEFT JOIN immunization_log
    ON patients.patient_id = immunization_log.patient_id

-- Collapse all rows into one summary row per age_group.
GROUP BY age_group

-- Filter the GROUPED results (not individual rows) to only show
-- age groups with more than 1 total dose recorded.
HAVING COUNT(*) > 1;
```

## Result

| age_group | total_doses |
|---|---|
| Pediatric | 3 |
| Adult | 2 |
| Senior | 2 |

*(The `Unknown` group, representing patient 106 with no immunization record, has only 1 row and is filtered out by the `HAVING` clause.)*

## Concepts Demonstrated

| Concept | Where it's used |
|---|---|
| `LEFT JOIN` | Keeps all patients, even those with no vaccination history |
| `CASE WHEN` | Converts raw age values into readable categories, including a `NULL` check |
| `GROUP BY` | Collapses individual dose records into one row per age group |
| `COUNT(*)` | Aggregates the number of doses within each group |
| `HAVING` | Filters the *grouped* results, which `WHERE` cannot do |

## Notes / Lessons Learned

- Every expression in a `SELECT` list — including multi-line `CASE` blocks — needs a comma after it, just like any other item in the list (except the last one before `FROM`).
- `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation. Aggregate functions like `COUNT()` don't exist yet at the row level, so `HAVING` is required to filter on them.
- A `LEFT JOIN` can produce `NULL` values in columns from the right-hand table — always plan for that when writing `CASE WHEN` logic downstream.

---

# Public Health SQL Portfolio: Flu Outreach Tracking- Full DDL → DML → DQL Lifecycle (August 13, 2026)

## Overview

This piece demonstrates the full lifecycle of a database table: **building it, populating it, and querying it.** It models a simple flu vaccination outreach log for a clinic — the kind of tracking system used to monitor whether patients contacted during an outreach campaign actually got vaccinated.

The three stages map to the three core SQL sublanguages:

| Stage | SQL Type | Purpose |
|---|---|---|
| 1. Build the table | **DDL** (Data Definition Language) | Define the table's structure and column types |
| 2. Add the data | **DML** (Data Manipulation Language) | Insert actual outreach records into the table |
| 3. Ask a question | **DQL** (Data Query Language) | Summarize vaccination outcomes from the data |

## Stage 1 — DDL: Build the Table

```sql
-- Create a new table to track flu outreach contacts.
-- vaccinated is stored as VARCHAR(3) since it only ever holds
-- 'Yes' or 'No' -- sized intentionally rather than oversized.
CREATE TABLE flu_outreach (
    contact_id INT,
    patient_last_name VARCHAR(50),
    contact_date DATE,
    vaccinated VARCHAR(3)
);
```

## Stage 2 — DML: Populate the Table

```sql
-- Insert outreach records. Columns are explicitly named to avoid
-- any ambiguity about which value maps to which column.
-- Text and date values are wrapped in single quotes; numbers are not.
INSERT INTO flu_outreach (contact_id, patient_last_name, contact_date, vaccinated)
VALUES 
    (101, 'James',      '2024-04-05', 'No'),
    (102, 'Tagoe',       '2024-04-29', 'Yes'),
    (103, 'Franchetti',  '2024-05-13', 'Yes'),
    (104, 'Zhang',       '2024-05-20', 'No');
```

Resulting table:

| contact_id | patient_last_name | contact_date | vaccinated |
|---|---|---|---|
| 101 | James | 2024-04-05 | No |
| 102 | Tagoe | 2024-04-29 | Yes |
| 103 | Franchetti | 2024-05-13 | Yes |
| 104 | Zhang | 2024-05-20 | No |

## Stage 3 — DQL: Query the Data

```sql
-- Count how many outreach contacts fall into each vaccination
-- status. No WHERE clause is used here on purpose -- filtering
-- to one status before grouping would defeat the point of
-- comparing Yes vs. No side by side.
SELECT vaccinated, COUNT(*) AS total_vaccinations
FROM flu_outreach
GROUP BY vaccinated;
```

## Result

| vaccinated | total_vaccinations |
|---|---|
| No | 2 |
| Yes | 2 |

## Concepts Demonstrated

| Concept | Stage | Purpose |
|---|---|---|
| `CREATE TABLE` | DDL | Defines the table and its column data types |
| `INSERT INTO ... VALUES` | DML | Adds multiple rows of real data in one statement |
| `GROUP BY` + `COUNT(*)` | DQL | Summarizes outcomes across the inserted data |

## Notes / Lessons Learned

- DDL, DML, and DQL aren't separate skills — they're three stages of the same lifecycle: **build the container, fill it, then ask questions of it.**
- `WHERE` filters individual rows; leaving it out here was intentional, since the goal was a side-by-side comparison across *all* categories, not a subset.
- Sizing a column's data type intentionally (`VARCHAR(3)` for a Yes/No field) is good practice — it documents the expected range of values right in the schema.

---

# CPM Performance Dashboard Data Prep Using Python/pandas/numpy (August 14, 2026)

## Overview

This project simulates a data-prep and reporting workflow for a psychiatric care setting, modeled on the kind of work outlined in a real Performance & Data Analyst job description: monitoring KPIs like length of stay and readmissions, validating data integrity, and communicating findings to leadership.

It walks through the full analyst workflow: **load data → explore it → check data integrity → create calculated fields → summarize KPIs → write up findings.**

Tools used: `pandas`, `numpy`

## Step 1: Load the Dataset

```python
import pandas as pd
import numpy as np

data = {
    'patient_id': [301, 302, 303, 304, 305, 306, 307, 308, 309, 310, 311, 312, 313, 314, 315],
    'unit': ['Adult Psych', 'Adolescent Psych', 'Geriatric Psych', 'Geriatric Psych', 'Adult Psych',
             'Adolescent Psych', 'Adolescent Psych', 'Geriatric Psych', 'Geriatric Psych', 'Adult Psych',
             'Adolescent Psych', 'Adolescent Psych', 'Geriatric Psych', 'Geriatric Psych', 'Adult Psych'],
    'length_of_stay_days': [7, 3, 12, 5, 9, 2, 6, np.nan, 14, 8, 1, 11, 13, 10, 4],
    'readmitted_status': ['Yes', 'Yes', 'No', 'Yes', 'No', 'No', 'No', 'Yes', 'No', 'Yes',
                           'Yes', 'Yes', 'No', 'No', 'Yes'],
    'discharge_disposition': ['Home', 'AMA', 'Home', 'Transfer', 'Transfer', 'Home', 'Home', 'AMA',
                               'AMA', 'Home', 'Transfer', 'AMA', 'Transfer', 'Home', 'Home']
}

df = pd.DataFrame(data)
df
```

## Step 2: Explore the Data + Check Data Integrity

```python
df.head()
df.info()
df.describe()
```

```python
# Check for missing values
df.isnull().sum()
```

`length_of_stay_days` came back with 1 missing value (patient 308). Rather than dropping that patient's record and losing their other data, the gap is filled with the column's average — a defensible, transparent way to handle a single missing data point in a small sample. Therefore, the missing value (np.nan) is 7.5.

```python
# Fill the missing value with the column mean
df['length_of_stay_days'] = df['length_of_stay_days'].fillna(df['length_of_stay_days'].mean())

# Confirm the gap is resolved
df.isnull().sum()
```

## Step 3: Create Calculated Columns

```python
# Bucket length of stay into categories
def categorize_stay(days):
    if days <= 3:
        return 'Short Stay'
    elif 4 <= days <= 8:
        return 'Moderate Stay'
    else:
        return 'Extended Stay'

df['stay_category'] = df['length_of_stay_days'].apply(categorize_stay)
```

```python
# Flag readmission risk
def readmission_flag(status):
    if status == 'Yes':
        return 'Risk'
    else:
        return 'Low Risk'

df['readmission_risk'] = df['readmitted_status'].apply(readmission_flag)

df
```

## Step 4: KPI Summaries


This metric counts the total volume of patient readmissions across the entire dataset. It establishes the overall baseline split between readmitted and non-readmitted individuals.

```python
# Overall readmission counts
df['readmitted_status'].value_counts()
```
| readmitted_status | count |
|---|---|
| Yes | 8 |
| No | 7 |


This aggregation calculates the mean length of stay (in days) grouped by specific psychiatric units. It highlights how hospitalization windows scale from younger cohorts to geriatric care.

```python
# Average length of stay by unit
df.groupby('unit')['length_of_stay_days'].mean()
```
| unit | avg length_of_stay_days |
|---|---|
| Adolescent Psych | 4.60 |
| Adult Psych | 7.00 |
| Geriatric Psych | 10.25 |


This segment breaks down patient risk stratifications within each specific care unit. It uses a secondary grouping to isolate high-risk versus low-risk volumes.

```python
# Readmission risk breakdown by unit
df.groupby('unit')['readmission_risk'].value_counts()
```
| unit | readmission_risk | count |
|---|---|---|
| Adolescent Psych | Risk | 3 |
| Adolescent Psych | Low Risk | 2 |
| Adult Psych | Risk | 3 |
| Adult Psych | Low Risk | 1 |
| Geriatric Psych | Low Risk | 4 |
| Geriatric Psych | Risk | 2 |

## Step 5: Summary of Findings

Across the sample of 15 patients, readmission rates were nearly evenly split, with 8 patients flagged as readmitted and 7 not. Average length of stay varied notably by unit: Geriatric Psych patients stayed longest (10.25 days on average), compared to Adult Psych (7.0 days) and Adolescent Psych (4.6 days). Notably, this pattern did not carry over to readmission risk — Geriatric Psych patients had the lowest proportion flagged as high-risk, while Adolescent and Adult Psych patients skewed higher-risk despite shorter average stays. This inverse relationship is worth flagging for further investigation, since it runs counter to the assumption that longer stays would correlate with higher readmission risk.

## Concepts Demonstrated

| Concept | pandas Tool | Purpose |
|---|---|---|
| Loading data | `pd.DataFrame()` | Building a table structure in memory |
| Exploring data | `.head()`, `.info()`, `.describe()` | Sanity-checking a new dataset before analysis |
| Data integrity | `.isnull().sum()`, `.fillna()` | Identifying and handling missing values |
| Calculated fields | Custom function + `.apply()` | Bucketing/flagging values, the pandas equivalent of SQL's `CASE WHEN` |
| KPI summaries | `.value_counts()`, `.groupby()` | Summarizing counts and averages by category, the pandas equivalent of `GROUP BY` |

## Notes / Lessons Learned

- Notebooks don't enforce top-to-bottom execution order — cells run in whatever order they're clicked, which can silently overwrite a variable like `df` with stale data from an earlier cell. `Runtime → Restart session` followed by `Runtime → Run all` is the reliable way to confirm a notebook actually runs cleanly start to finish, which matters both for debugging and for reproducibility when someone else opens the notebook.
- Leftover troubleshooting/test cells (built to isolate one piece of logic) should be deleted once they've served their purpose — otherwise they can silently re-run and clobber real work further down the notebook.
- `.fillna()` and similar data-cleaning decisions are analytical choices worth documenting, not just technical steps — particularly relevant in a regulatory/compliance-facing role where data integrity is closely scrutinized.
