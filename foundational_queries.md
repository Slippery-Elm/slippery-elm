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
