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
