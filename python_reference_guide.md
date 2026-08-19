# Python for Public Health Data Analysis: pandas, numpy & pyreadstat Reference Guide 🐍
 
A personal reference guide covering the core components of `pandas`, `numpy`, and `pyreadstat` — built while learning Python for public health data analysis and health informatics work.
 
---
 
## Table of Contents 🌎
1. [Setup](#1-setup)
2. [Loading Data](#2-loading-data)
3. [Exploring a DataFrame](#3-exploring-a-dataframe)
4. [Selecting Data](#4-selecting-data)
5. [Missing Data](#5-missing-data)
6. [Grouping & Aggregating](#6-grouping--aggregating)
7. [Creating Calculated Columns](#7-creating-calculated-columns)
8. [Joining Two DataFrames](#8-joining-two-dataframes)
9. [Common Gotchas](#9-common-gotchas)
10. [Side-by-Side Example Scripts](#10-side-by-side-example-scripts)
---
 
## 1. Setup ⚙️
 
Import all three libraries before use. `pandas` and `numpy` come pre-installed in Google Colab; `pyreadstat` needs to be installed separately.
 
```python
# Standard imports for a data analysis session
import pandas as pd
import numpy as np
 
# pyreadstat is not pre-installed — install it first (only needs to be done once per session)
!pip install pyreadstat
import pyreadstat
```
 
**What each library is for:**
- **pandas** — the core tool for working with tables (DataFrames): loading, filtering, grouping, summarizing.
- **numpy** — numerical operations under the hood; pandas is actually built on top of numpy. Most commonly used directly for `np.nan` (missing values) and `np.where()` (conditional logic).
- **pyreadstat** — reads statistical software file formats (SAS `.sas7bdat`, SPSS `.sav`, Stata `.dta`) directly into a pandas DataFrame, since these formats aren't plain text like CSV.
---
 
## 2. Loading Data ⏳
 
### pandas — CSV
```python
df = pd.read_csv('brfss_2017_birmingham.csv')
```
 
### pandas — Excel
```python
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')
```
 
### pandas — building a DataFrame manually (for mock/practice data)
```python
data = {
    'respondent_id': [1001, 1002, 1003],
    'condition': ['Asthma', 'Diabetes', 'Asthma']
}
df = pd.DataFrame(data)
```
 
### pyreadstat — SAS files
```python
df, meta = pyreadstat.read_sas7bdat('brfss_2017.sas7bdat')
```
Note the **two return values**: `df` is the actual data, and `meta` is a separate object holding metadata — things like each variable's original label, value labels (e.g., `1` = "Yes", `2` = "No"), and formats. This is a key difference from `pd.read_csv()`, which only returns one thing.
 
### pyreadstat — SPSS files
```python
df, meta = pyreadstat.read_sav('brfss_2017.sav')
```
 
### pyreadstat — Stata files
```python
df, meta = pyreadstat.read_dta('brfss_2017.dta')
```
 
### Viewing metadata from a pyreadstat file
```python
print(meta.column_names)        # list of raw variable names
print(meta.column_labels)       # human-readable labels for each variable
print(meta.variable_value_labels)  # dict mapping coded values to their meaning (e.g., {'SEX': {1: 'Male', 2: 'Female'}})
```
This is genuinely useful for BRFSS-style datasets, where columns often use cryptic names (`_RFHLTH`) and coded numeric values that need a lookup to interpret.
 
---
 
## 3. Exploring a DataFrame 🔍
 
```python
df.head()        # first 5 rows (df.head(10) for first 10)
df.tail()        # last 5 rows
df.shape         # (rows, columns) as a tuple — no parentheses, this is an attribute, not a method
df.columns       # list of column names
df.dtypes        # data type of each column
df.info()        # combined summary: columns, non-null counts, dtypes, memory usage
df.describe()    # summary statistics for numeric columns only (count, mean, std, min, 25/50/75%, max)
df.describe(include='object')  # summary stats for text/categorical columns instead (count, unique, top, freq)
df.nunique()     # number of unique values per column — useful for spotting categorical vs. continuous columns
df.sample(5)     # 5 random rows — useful for a less biased "peek" than always seeing the top rows
```
 
---
 
## 4. Selecting Data 👉🏾
 
### Selecting one column (returns a Series)
```python
df['condition']
```
 
### Selecting multiple columns (returns a DataFrame — note the double brackets)
```python
df[['respondent_id', 'condition']]
```
 
### Selecting by row position — `.iloc[]`
```python
df.iloc[0]        # first row
df.iloc[0:3]      # first 3 rows
df.iloc[0, 1]     # row 0, column 1 (by position)
```
 
### Selecting by row/column label — `.loc[]`
```python
df.loc[0]                      # row with index label 0
df.loc[0:3]                    # rows with index labels 0 through 3 (inclusive!)
df.loc[0, 'condition']         # row 0, column 'condition' (by label)
df.loc[df['county'] == 'Jefferson', 'condition']  # filter rows AND select a column in one line
```
 
### Filtering rows (the `WHERE` equivalent)
```python
df[df['age'] > 50]
df[df['county'] == 'Jefferson']
df[df['condition'].isin(['Asthma', 'Diabetes'])]   # equivalent to SQL's WHERE condition IN (...)
```
 
### Combining conditions (the `AND` / `OR` equivalent)
```python
df[(df['county'] == 'Jefferson') & (df['age'] > 50)]   # AND
df[(df['condition'] == 'Asthma') | (df['condition'] == 'Diabetes')]   # OR
```
Each condition needs its own parentheses — this is a strict rule, not a style choice.
 
---
 
## 5. Missing Data ❌
 
```python
df.isnull()             # DataFrame of True/False, one per cell
df.isnull().sum()       # count of missing values per column
df.isnull().sum().sum() # total missing values in the entire DataFrame
df.notnull()            # opposite of isnull() — True where data IS present
 
df.dropna()                          # drop any row with at least one missing value
df.dropna(subset=['age'])            # only drop rows missing THIS specific column
df.dropna(axis=1)                    # drop columns (not rows) that have any missing values
 
df['age'] = df['age'].fillna(df['age'].mean())    # fill with the column mean
df['age'] = df['age'].fillna(df['age'].median())  # fill with the median (better than mean if data is skewed)
df['condition'] = df['condition'].fillna('Unknown')  # fill a text column with a placeholder
```
 
`np.nan` (from numpy) is pandas' standard way of representing a missing value — it's the pandas/Python equivalent of SQL's `NULL`.
 
---
 
## 6. Grouping & Aggregating 👥
 
```python
df['condition'].value_counts()                          # count of each unique value in one column
df['condition'].value_counts(normalize=True)             # same, but as proportions (0.0–1.0) instead of raw counts
 
df.groupby('condition')['age'].mean()                     # average age per condition
df.groupby('condition')['age'].agg(['count', 'mean', 'min', 'max'])  # multiple stats at once
df.groupby(['county', 'condition'])['age'].mean()          # group by MORE than one column at once
 
df.groupby('condition').size()                             # row count per group (similar to COUNT(*))
df.groupby('condition')['respondent_id'].count()            # count of non-null values in one specific column
```
 
---
 
## 7. Creating Calculated Columns 📊
 
### Simple True/False column
```python
df['is_senior'] = df['age'] >= 65
```
 
### Two-category column with `np.where()`
```python
df['age_bracket'] = np.where(df['age'] >= 65, 'Senior', 'Non-Senior')
```
 
### Three or more categories with a custom function + `.apply()`
```python
def age_group(age):
    if age < 18:
        return 'Pediatric'
    elif age < 65:
        return 'Adult'
    else:
        return 'Senior'
 
df['age_group'] = df['age'].apply(age_group)
```
 
### Calculated column from math on other columns
```python
df['bmi'] = df['weight_kg'] / (df['height_m'] ** 2)
```
 
---
 
## 8. Joining Two DataFrames ⛓️
 
Pandas' `.merge()` is the equivalent of SQL's `JOIN`. The `how` parameter controls join type — same four options as SQL:
 
```python
pd.merge(df1, df2, on='respondent_id', how='inner')   # only matching rows from both — same as SQL INNER JOIN
pd.merge(df1, df2, on='respondent_id', how='left')    # all of df1, matched or not — same as SQL LEFT JOIN
pd.merge(df1, df2, on='respondent_id', how='right')   # all of df2, matched or not — same as SQL RIGHT JOIN
pd.merge(df1, df2, on='respondent_id', how='outer')   # everything from both — same as SQL FULL JOIN
```
 
If the key column has different names in each DataFrame:
```python
pd.merge(df1, df2, left_on='respondent_id', right_on='patient_id', how='left')
```
 
Combining more than two DataFrames — merge twice:
```python
combined = pd.merge(df1, df2, on='respondent_id', how='left')
combined = pd.merge(combined, df3, on='respondent_id', how='left')
```
 
---
 
## 9. Common Gotchas 🤔
 
- **`=` vs `==`** — a single `=` assigns a value; `==` checks equality. Using `=` inside a filter condition causes an error, not a silent bug.
- **Curly/smart quotes** — copying code from Word, Notes, or certain websites can silently swap straight quotes (`'`) for curly ones (`’`), which Python cannot parse as strings. If a line throws a confusing syntax error despite looking correct, retype the quotation marks manually rather than pasting them.
- **Double brackets for multiple columns** — `df['col']` returns a Series; `df[['col']]` (double brackets) returns a DataFrame. Selecting multiple columns always requires the double-bracket list syntax: `df[['col1', 'col2']]`.
- **Cell execution order in Colab/Jupyter** — cells run in whatever order you click "Run" on them, not necessarily top-to-bottom page order. Re-running an old cell can silently overwrite `df` with stale data. Use `Runtime → Restart session` then `Runtime → Run all` to confirm a notebook actually runs cleanly start to finish.
- **Leftover troubleshooting cells** — a cell built to test one small piece of logic in isolation can keep re-running and silently overwriting real work further down the notebook. Delete test cells once they've done their job.
- **`.loc[]` is inclusive on both ends; `.iloc[]` is not** — `df.loc[0:3]` returns rows 0, 1, 2, AND 3 (4 rows). `df.iloc[0:3]` returns only rows 0, 1, 2 (3 rows) — same "up to but not including" behavior as most Python slicing.
- **Chained aggregate methods must match the data type** — running `.mean()` on a text column, or `.value_counts()` directly on a full DataFrame instead of a single column, will error or behave unexpectedly.
- **`pyreadstat` returns two objects, not one** — forgetting to unpack both (`df, meta = ...`) will assign the whole tuple to `df`, breaking every later step.
---
 
## 10. Side-by-Side Example Scripts 📜
 
Same task, shown three ways, using a small mock BRFSS-style dataset.
 
### Setup (shared across all three examples)
```python
import pandas as pd
import numpy as np
 
data = {
    'respondent_id': [1001, 1002, 1003, 1004, 1005, 1006],
    'county': ['Jefferson', 'Jefferson', 'Shelby', 'Jefferson', 'Shelby', 'Jefferson'],
    'condition': ['Asthma', 'Diabetes', 'Asthma', 'Diabetes', 'Diabetes', 'Asthma'],
    'age': [24, 45, 67, np.nan, 52, 19]
}
df = pd.DataFrame(data)
```
 
### Example A — pandas: Full data-prep pass
```python
# Explore
df.info()
df.isnull().sum()
 
# Handle missing age
df['age'] = df['age'].fillna(df['age'].mean())
 
# Calculated column
def age_group(age):
    if age < 18:
        return 'Pediatric'
    elif age < 65:
        return 'Adult'
    else:
        return 'Senior'
 
df['age_group'] = df['age'].apply(age_group)
 
# KPI summary
df.groupby('condition')['age'].mean()
df.groupby(['county', 'condition']).size()
```
 
### Example B — numpy: Where numpy shows up directly in a pandas workflow
```python
# np.nan used to represent a missing value manually
df.loc[6] = [1007, 'Shelby', 'Asthma', np.nan]
 
# np.where() for a simple two-category calculated column
df['is_senior'] = np.where(df['age'] >= 65, 'Senior', 'Non-Senior')
 
# numpy functions work directly on a pandas column since a Series is built on a numpy array
print(np.mean(df['age']))
print(np.max(df['age']))
```
 
### Example C — pyreadstat: Loading a real SAS file and using its metadata
```python
import pyreadstat
 
# Load a SAS file — note the two return values
df, meta = pyreadstat.read_sas7bdat('brfss_2017.sas7bdat')
 
# Inspect what the coded columns actually mean before analyzing
print(meta.column_labels)
print(meta.variable_value_labels.get('_STATE'))  # e.g., {1: 'Alabama', 2: 'Alaska', ...}
 
# From here, the rest of the workflow is identical to Examples A and B —
# once loaded, a pyreadstat DataFrame behaves exactly like any other pandas DataFrame
df.info()
df.groupby('_STATE').size()
```
 
---
 
## Quick-Reference: SQL → pandas Mapping 🗺️
 
| SQL | pandas |
|---|---|
| `SELECT col FROM table` | `df['col']` |
| `SELECT col1, col2 FROM table` | `df[['col1', 'col2']]` |
| `WHERE col = 'x'` | `df[df['col'] == 'x']` |
| `WHERE col > x AND col2 = 'y'` | `df[(df['col'] > x) & (df['col2'] == 'y')]` |
| `GROUP BY col, COUNT(*)` | `df['col'].value_counts()` or `df.groupby('col').size()` |
| `GROUP BY col, AVG(x)` | `df.groupby('col')['x'].mean()` |
| `INNER JOIN` | `pd.merge(df1, df2, on='key', how='inner')` |
| `LEFT JOIN` | `pd.merge(df1, df2, on='key', how='left')` |
| `CASE WHEN ... END` | Custom function + `.apply()`, or `np.where()` for two categories |
| `col IS NULL` | `df['col'].isnull()` |
