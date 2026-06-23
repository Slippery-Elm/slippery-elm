# Public Health Data Analytics Portfolio

## Project 1: Chronic Disease Population Health Dashboard
**Track:** Healthcare Analytics  
**Data Source:** Mock BRFSS Chronic Condition Data  

### Objectives
Identify regional wellness zones by isolating patients who are sedentary but currently free of diagnosed cardiovascular disease. This allows healthcare administrators to implement preemptive preventative health interventions.

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
