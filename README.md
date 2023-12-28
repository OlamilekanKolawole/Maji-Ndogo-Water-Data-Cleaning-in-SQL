# Maji-Ndogo-Water-Data-Cleaning-in-SQL
## Data Cleaning of Maji Ndogo Water in SQL Server

## Table of Contents

- [Project Overview](#project-overview)

- [Data Sources](#data-sources)

- [Tools](#tools)

- [Data Cleaning and Preparation](#data-cleaning-and-preparation)

- [Exploratory Data Analysis](exploratory-data-analysis)

- [Data Analysis](data-analysis)

- [Result and Findings](#result-and-findings)

- [References](#references)

- [Limitations](#limitations)

- [Recommendations](#recommendations)

### Project Overview 
This Data Analysis Project aims to provide meaningful insights to Maji Ndogwo Water Crisis, By analysing and exploring Database structure,We seek to identify trends, make data driven recommendations  and gain a deeper understanding of Maji Ndogo Water Crisis.

### Data Sources 
Maji Ndogo water services: The primary Dataset used for this analysis is the "md_water_services.SQL" Text File, containing detailed information about Maji Ndogo Water issues.

### Tools

- Excel - Data Cleaning [Download Here](https://microsoft.com)
- MySQL Workbench - Data Analysis [Download Here](https://dev.mysql.com/downloads/workbench/)
- PowerBI - Creating Reports [Download Here](https://www.microsoft.com/en-us/download/details.aspx?id=58494)

### Data Cleaning and Preparation 
In the initial data preparation phase, we performed the following Tasks 
1. Data Loading and inspection.
2. Handling missing values.
3. Data cleaning and formatting.

### Exploratory Data Analysis 
EDA involved exploring the Maji Ndogo water services Data to answer key questions, Such as:
-  What is the address of Bello Azibo?
 ```sql
SELECT *
FROM employee WHERE employee_name = 'Bello Azibo';
```

-  What is the name and phone number of our Microbiologist?
  ```sql 
  SELECT *
FROM employee WHERE position = 'Micro Biologist';
 ```
-  What is the source id of the water source shared by the most number of people?
```sql
  SELECT max(number_of_people_served)
FROM water_source;
```

-  What is the population of Maji Ndogo?
```sql
SELECT * FROM md_water_services.global_water_access Where name = 'Maji Ndogo';
```

-  Which SQL query returns records of employees who are Civil Engineers residing in Dahabu
or living on an avenue?

```sql
SELECT * FROM md_water_services.employee where position = 'Civil Engineer' AND
(town_name = 'Dahabu' Or '%avenue%');
```
-  Create a query to identify potentially suspicious field workers based on an anonymous tip.
  This is the description we are given:
  The employee's phone number contained the digits 86 or 11.
  The employee's last name started with either an A or an M.
  The employee was a Field Surveyor
```sql
SELECT
*
FROM
employee
WHERE
(phone_number LIKE '%86%'
OR Phone_number LIKE '%11%')
AND (employee_name LIKE '% A%'
OR employee_name LIKE '% M%')
AND Position = 'Field Surveyor';
```

-  Which query will identify the records with a quality score of 10, visited more than once?
```sql
SELECT * FROM water_quality WHERE visit_count >='2' AND subjective_quality_score = '10';
```
-  Correct the phone number for the employee named'Bello Azibo'. The correct number is +99643864786.

  ### Data Analysis

  ### Beginning Our Data-Driven Journey in Maji Ndogo (Level 1)

#### looking at the Location table

```sql
SELECT 
    *
FROM
    location
LIMIT 5; 
```

#### looking at the Visit table
```sql
SELECT 
    *
FROM
    visits
LIMIT 5;
```


#### looking at the Water Source table
```sql
SELECT 
    *
FROM
    water_source
LIMIT 5;
```


#### looking at the unique types of water sources
```sql
SELECT 
    DISTINCT type_of_water_source
FROM
    water_source
LIMIT 5;
```


#### query that retrieves all records from this table where the time_in_queue is more than some crazy time, say 500 min. How would it feel to queue 8 hours for water?
```sql
SELECT 
    *
FROM
    visits
WHERE time_in_queue > 500;
```

```sql
SELECT 
    *
FROM
    visits
WHERE time_in_queue > 500 AND source_id IN ('SoRu35083224','AmRu14089224','HaRu19601224','AkLu02523224');
```

```sql
SELECT 
    *
FROM
    water_source
WHERE
    source_id IN ('SoRu35083224' , 'AmRu14089224',
        'HaRu19601224',
        'AkLu02523224');
```
### query to find records where the subject_quality_score is 10 -- only looking for home taps -- and where the source
was visited a second time.

```sql        
SELECT 
    *
FROM
    water_quality
WHERE
    subjective_quality_score = 10
        AND visit_count = 2;
```
#### Looking at the first five row of well polluttion 
```sql
SELECT 
    *
FROM
    well_pollution
LIMIT 5;
```


#### write a query that checks if the results is Clean but the biological column is > 0.01
```sql
SELECT 
    *
FROM
    well_pollution
WHERE
    results = 'Clean' AND biological > 0.01;
```

#### The Data were recorded wrongly as a result column was not supposed to be tagged Clean if the biogical column is > 0.01 

```sql
SELECT 
    *
FROM
    well_pollution
WHERE
    results = 'Clean' AND biological > 0.01 AND description LIKE 'Clean_%';
```

#### Now we have to replace them in their correct order by creat a table we can test query in first before executing on our main table 

```sql
Create Table
md_water_services.well_pollution_copy
AS (
SELECT 
    *
FROM
    well_pollution
);
```


```sql
SELECT * FROM md_water_services.well_pollution
WHERE results = 'Clean' AND biological > 0.01 AND description LIKE 'Clean_%';
```

```sql
SET SQL_SAFE_UPDATES = 0;
UPDATE
well_pollution_copy
SET
description = 'Bacteria: E. coli'
WHERE
description = 'Clean Bacteria: E. coli';
UPDATE
well_pollution_copy
SET
description = 'Bacteria: Giardia Lamblia'
WHERE
description = 'Clean Bacteria: Giardia Lamblia';
UPDATE
well_pollution_copy
SET
results = 'Contaminated: Biological'
WHERE
biological > 0.01 AND results = 'Clean';
```
 #### Now to execute on the Main Table, We use this

 ```sql
SET SQL_SAFE_UPDATES = 0;
UPDATE
well_pollution
SET
description = 'Bacteria: E. coli'
WHERE
description = 'Clean Bacteria: E. coli';
UPDATE
well_pollution
SET
description = 'Bacteria: Giardia Lamblia'
WHERE
description = 'Clean Bacteria: Giardia Lamblia';
UPDATE
well_pollution
SET
results = 'Contaminated: Biological'
WHERE
biological > 0.01 AND results = 'Clean';
```




    
