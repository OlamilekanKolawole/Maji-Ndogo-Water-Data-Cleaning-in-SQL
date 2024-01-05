# Maji-Ndogo-Water-Data-Cleaning-in-SQL
## Data Cleaning of Maji Ndogo Water in SQL Server

## Table of Contents

- [Project Overview](#project-overview)

- [Data Sources](#data-sources)

- [Tools](#tools)

- [Data Cleaning and Preparation](#data-cleaning-and-preparation)

- [Exploratory Data Analysis](#exploratory-data-analysis)

- [Data Analysis](#data-analysis)

- [Result and Findings](#result-and-findings)

- [References](#references)

- [Limitations](#limitations)

- [Recommendations](#recommendations)

### Project Overview 
This Data Analysis Project aims to provide meaningful insights to Maji Ndogwo Water Crisis, By analysing and exploring Database structure,We seek to identify trends, make data driven recommendations  and gain a deeper understanding of Maji Ndogo Water Crisis.

![Sokoto river](https://github.com/OlamilekanKolawole/Maji-Ndogo-Water-Data-Cleaning-in-SQL/assets/151407380/15955b1a-b35e-47a6-871c-cf2b449199ed)


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

  ## Data Analysis

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

## Clustering data to unveil Maji Ndogo's water  (Level 2)

### Looking at the employee Data we nothing the Email was not given However,We will have to send them reports and figures, so let's update it

#### First up, let's remove the space between the first and last names using REPLACE(). 
```sql
SELECT 
    REPLACE(employee_name, ' ', '.')         −− Replace the space with a full stop 
FROM
    employee;
 ```

#### Then we can use LOWER() with the result we just got. Now the name part is correct.
```sql
   SELECT 
    LOWER(REPLACE(employee_name, ' ', '.'))     −− Make it all lower case
FROM
    employee;
```

#### We then use CONCAT() to add the rest of the email address:
```sql
    SELECT 
    CONCAT(LOWER(REPLACE(employee_name, ' ', '.')),      add it all together     
            '@ndogowater.gov') AS new_email
FROM
    employee;
```


####  Now we can UPDATE IT
```sql
SET SQL_SAFE_UPDATES = 0;    
UPDATE employee 
SET 
    email = CONCAT(LOWER(REPLACE(employee_name, ' ', '.')),
            '@ndogowater.gov');
```

####  The phone numbers should be 12 characters long, consisting of the plus sign, area code (99), and the phone number digits. However, when we use the LENGTH(column) function, it returns 13 characters, indicating there's an extra character.

```sql
SELECT 
    LENGTH(phone_number)
FROM
    employee;
```


#### That's because there is a space at the end of the number! If you try to send an automated SMS to that number it will fail. This happens so often that they create a function

####  Now to remove any leading or trailing spaces from a string, we use 

```sql
SELECT 
    TRIM(phone_number)
FROM
    employee;
```

####  After Trimming we confirmed to see if the digits are 12 now before updating ,Lets check

```sql
    SELECT 
    length((phone_number))   ----Now we have 12 digits, so we can update 
FROM
    employee;
```

```sql
SET SQL_SAFE_UPDATES = 0;
UPDATE employee 
SET 
    phone_number = TRIM(phone_number); --- UPDATED
```



####  Use the employee table to count how many of our employees live in each town

```sql
SELECT 
    town_name, COUNT(employee_name) AS num_of_employee
FROM
    employee
GROUP BY town_name;
```


####  Pres. Naledi has asked we send out an email or message congratulating the top 3 field surveyors. So let's use the database to get the employee_ids and use those to get the names, email and phone numbers of the three field surveyors with the most location visits

```sql
SELECT 
    assigned_employee_id AS Employee_ID,
    COUNT(visit_count) AS Number_of_visits
FROM
    visits
GROUP BY assigned_employee_id
ORDER BY Number_of_visits DESC
LIMIT 3;
```

####  Now we need there info so tHE PRESIDENT can reach out to them

```sql
SELECT 
    assigned_employee_id, employee_name, email, phone_number
FROM
    employee
WHERE
    assigned_employee_id IN ('1' , '30', '34');
```


####  Create a query that counts the number of records per town

```sql
SELECT 
    COUNT(location_type) AS records_per_town,town_name
FROM
    location
GROUP BY town_name;
```


####  OR

```sql
SELECT 
    COUNT(town_name) AS records_per_town,town_name
FROM
    location
GROUP BY town_name;
```


####  Now count the records per province.

```sql
SELECT 
    province_name, COUNT(province_name) AS records_per_province
FROM
    location
GROUP BY province_name;
```



####  1. Create a result set showing:
- province_name
- town_name
- An aggregated count of records for each town (consider naming this records_per_town).
- Ensure your data is grouped by both province_name and town_name.
2. Order your results primarily by province_name. Within each province, further sort the towns by their record counts in descending order.

```sql
SELECT 
    province_name,
    town_name,
    COUNT(town_name) AS records_per_town
FROM
    location
GROUP BY province_name , town_name
ORDER BY province_name DESC ;
```



#### Finally, look at the number of records for each location type

```sql
SELECT 
    location_type, COUNT(location_type) AS num_sources
FROM
    location
GROUP BY location_type;
```


We can see that there are more rural sources than urban, but it's really hard to understand those numbers. Percentages are more relatable.
If we use SQL as a very overpowered calculator:

```sql
SELECT 23740 / (15910 + 23740) * 100
```



## WATER SOURCE TABLE 

#### The way I look at this table; we have access to different water source types and the number of people using each source.
#### These are the questions that I am curious about.
#### 1. How many people did we survey in total?
```sql
SELECT 
    type_of_water_source,
    COUNT(number_of_people_served) AS Total_People_served
FROM
    water_source
GROUP BY type_of_water_source , number_of_people_served
ORDER BY Total_People_served DESC;
```


#### 2. How many wells, taps and rivers are there?
```sql
SELECT 
    type_of_water_source,
    COUNT(type_of_water_source) AS number_of_sources
FROM
    water_source
GROUP BY type_of_water_source 
ORDER BY number_of_sources;
```


#### 3. How many people share particular types of water sources on average?
```sql
SELECT 
    type_of_water_source,
    ROUND(AVG(number_of_people_served)) AS Avg_number_of_people_served
FROM
    water_source
GROUP BY type_of_water_source
ORDER BY Avg_number_of_people_served;
```



#### 4. How many people are getting water from each type of source?
```sql
SELECT 
    type_of_water_source,
    SUM(number_of_people_served) AS Total_people_served
FROM
    water_source
GROUP BY type_of_water_source
ORDER BY Total_people_served DESC;
```

#### These results are telling us that 644 people share a tap_in_home on average. Does that make sense?
#### No it doesn’t, right?
#### Remember I told you a few important things that apply to tap_in_home and broken_tap_in_home? The surveyors combined the data of many households together and added this as a single tap record, but each household actually has its own tap. In addition to this, there is an average of 6 people living in a home. So 6 people actually share 1 tap (not 644)


#### This means that 1 tap_in_home actually represents 644 ÷ 6 = ± 100 taps



#### It's a little hard to comprehend these numbers, but you can see that one of these is dominating. To make it a bit simpler to interpret, let's use percentages. First, we need the total number of citizens then use the result of that and divide each of the SUM(number_of_people_served) by that number, times 100, to get percentages.

```sql
SELECT 
    type_of_water_source,
    SUM(number_of_people_served) AS Total_people_served,
    ROUND((SUM(number_of_people_served) / (SELECT 
                    SUM(number_of_people_served)
                FROM
                    water_source)) * 100,
            0) AS Percentage_Served
FROM
    water_source
GROUP BY type_of_water_source
ORDER BY Total_people_served DESC;
```

#### By adding tap_in_home and tap_in_home_broken together, we see that 31% of people have water infrastructure installed in their homes, but 45% (14/31) of these taps are not working! This isn't the tap itself that is broken, but rather the infrastructure like treatment plants, reservoirs, pipes, and pumps that serve these homes that are broken. 18% of people are using wells. But only 4916 out of 17383 are clean = 28% (from last week).





### Start of a solution

#### At some point, we will have to fix or improve all of the infrastructure, so we should start thinking about how we can make a data-driven decision
how to do it. I think a simple approach is to fix the things that affect most people first. So let's write a query that ranks each type of source based
on how many people in total use it. RANK() should tell you we are going to need a window function to do this, so let's think through the problem.


#### We will need the following columns:
- Type of sources -- Easy
- Total people served grouped by the types -- We did that earlier, so that's easy too.
- A rank based on the total people served, grouped by the types -- A little harder.

```sql
SELECT type_of_water_source, 
Total_people_served,
RANK() OVER (ORDER BY Total_people_served DESC) AS rank_by_population
FROM (SELECT type_of_water_source, SUM(number_of_people_served) AS Total_people_served FROM water_source
GROUP BY type_of_water_source) AS Subquery ORDER BY Total_people_served DESC;
```



#### Ok, so we should fix shared taps first, then wells, and so on. But the next question is, which shared taps or wells should be fixed first? We can use the same logic; the most used sources should really be fixed first.

#### So create a query to do this, and keep these requirements in mind:
#### 1. The sources within each type should be assigned a rank.
#### 2. Limit the results to only improvable sources.
#### 3. Think about how to partition, filter and order the results set.
#### 4. Order the results to see the top of the list.

### USING RANK()

```sql
SELECT 
source_id,
type_of_water_source,
SUM(number_of_people_served) AS Total_people_served,
RANK() OVER (partition by type_of_water_source ORDER BY 
SUM(number_of_people_served)DESC) AS priority_rank
FROM water_source
group by source_id, type_of_water_source;
```


### USING DENSE RANK 

```sql
SELECT 
source_id,
type_of_water_source,
SUM(number_of_people_served) AS Total_people_served,
DENSE_RANK() OVER (partition by type_of_water_source ORDER BY 
SUM(number_of_people_served)DESC) AS priority_rank
FROM water_source
group by source_id, type_of_water_source;
```


### USING ROW NUMBER 

```sql
SELECT 
source_id,
type_of_water_source,
SUM(number_of_people_served) AS Total_people_served,
ROW_NUMBER() OVER (partition by type_of_water_source ORDER BY 
SUM(number_of_people_served)DESC) AS priority_rank
FROM water_source
group by source_id, type_of_water_source;
```





## Analysing queues

### Ok, these are some of the things I think are worth looking at:
#### 1. How long did the survey take?

```sql
SELECT 
    DATEDIFF(MAX(time_of_record), MIN(time_of_record)) AS Survey_Duration_in_Days
FROM
    visits;
```

#### 2. What is the average total queue time for water?
```sql
SELECT 
    AVG(time_in_queue) AS Average_Time_in_Queue
FROM
    visits;
```

#### 3. What is the average queue time on different days?

```sql
SELECT 
    DAYOFWEEK(time_of_record) AS Day_of_week,
    AVG(time_in_queue) AS Average_Queue_Time
FROM
    visits
GROUP BY Day_of_week
ORDER BY Average_Queue_Time DESC;
```


#### 4. Which day of the week has the most visits?

```sql
SELECT 
    DAYOFWEEK(time_of_record) AS Day_of_week,
    COUNT(visit_count) AS Visits_Count
FROM
    visits
GROUP BY Day_of_week
ORDER BY Visits_Count DESC
LIMIT 1;
```

#### OR  

```sql
SELECT 
    DAYOFWEEK(time_of_record) AS Day_of_week,
    COUNT(*) AS Visits_Count
FROM
    visits
GROUP BY Day_of_week
ORDER BY Visits_Count DESC
LIMIT 1;
```


#### 5.What is the overall trend in the number of visits over time ?

```sql
SELECT 
    DATE(time_of_record) AS Visit_Date,
    COUNT(*) AS Visits_Count
FROM
    visits
GROUP BY Visit_Date
ORDER BY Visit_Date;
```


#### 6.  How long people have to queue on average in Maji Ndogo
The ones who have tap at home where not included here and were recorded as zero on the time_in_queue

```sql
SELECT 
    AVG(NULLIF(time_in_queue, 0)) AS Average_Queue_Time
FROM
    visits;
```

#### 7. The queue times aggregated across the different days of the week.

```sql
SELECT 
    DAYNAME(time_of_record) AS Day_of_Week,
    ROUND(AVG(NULLIF(time_in_queue, 0))) AS Average_Queue_Time
FROM
    visits
GROUP BY Day_of_Week
ORDER BY CASE
    WHEN Day_of_Week = 'Monday' THEN 1
    WHEN Day_of_Week = 'Tuesday' THEN 2
    WHEN Day_of_Week = 'Wednesday' THEN 3
    WHEN Day_of_Week = 'Thursday' THEN 4
    WHEN Day_of_Week = 'Friday' THEN 5
    WHEN Day_of_Week = 'Saturday' THEN 6
    WHEN Day_of_Week = 'Sunday' THEN 7
END;
```
    

#### 8. Time during the day people collect water

```sql
SELECT 
    EXTRACT(HOUR FROM time_of_record) AS hour_of_day,
    ROUND(AVG(time_in_queue)) AS avg_queue_time
FROM
    visits
GROUP BY hour_of_day
ORDER BY hour_of_day;
```

To format the Time into a specific display format using TIME FORMAT 

```sql
SELECT 
    TIME_FORMAT(TIME(time_of_record), '%H:00') AS hour_of_day,
    ROUND(AVG(time_in_queue)) AS avg_queue_time
FROM
    visits
GROUP BY hour_of_day
ORDER BY hour_of_day;
```



```sql
SELECT 
    TIME_FORMAT(TIME(time_of_record), '%H:00') AS hour_of_day,
    DAYNAME(time_of_record),
    CASE
        WHEN DAYNAME(time_of_record) = 'Sunday' THEN time_in_queue
        ELSE NULL
    END AS Sunday
FROM
    visits
WHERE
    time_in_queue != 0;
```


### Making a Pivot Table on SQL 

#### 9. To aggregate by the hour, we can group the data by hour_of_day, and to make the table chronological, we can also order by hour_of_day. (For each day)

```sql
SELECT 
    TIME_FORMAT(TIME(time_of_record), '%H:00') AS hour_of_day,
    ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Sunday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Sunday,
    ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Monday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Monday,
    ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Tuesday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Tuesday,
    ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Wednesday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Wednesday,
    ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Thursday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Thursday,
    ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Friday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Friday,
    ROUND(AVG(CASE
                WHEN DAYNAME(time_of_record) = 'Saturday' THEN time_in_queue
                ELSE NULL
            END),
            0) AS Saturday
FROM
    visits
WHERE
    time_in_queue != 0
GROUP BY hour_of_day
ORDER BY hour_of_day;
```

See if you can spot these patterns:
1. Queues are very long on a Monday morning and Monday evening as people rush to get water.
2. Wednesday has the lowest queue times, but long queues on Wednesday evening.
3. People have to queue pretty much twice as long on Saturdays compared to the weekdays. It looks like people spend their Saturdays queueing
for water, perhaps for the week's supply?
4. The shortest queues are on Sundays, and this is a cultural thing. The people of Maji Ndogo prioritise family and religion, so Sundays are spent
with family and friends.


#### 10. Query for average of Sat, Sun, Tuesday
    
```sql
SELECT
CASE
WHEN DAYNAME(time_of_record) = 'Saturday' AND HOUR(time_of_record) = 12 THEN
'Saturday 12:00-13:00'
WHEN DAYNAME(time_of_record) = 'Tuesday' AND HOUR(time_of_record) = 18 THEN
'Tuesday 18:00-19:00'
WHEN DAYNAME(time_of_record) = 'Sunday' AND HOUR(time_of_record) = 9 THEN
'Sunday 09:00-10:00'
END AS Time_Period,
ROUND(AVG(time_in_queue), 0) AS Average_Queue_Time
FROM
visits
WHERE
(DAYNAME(time_of_record) = 'Saturday' AND HOUR(time_of_record) = 12) OR
(DAYNAME(time_of_record) = 'Tuesday' AND HOUR(time_of_record) = 18) OR
(DAYNAME(time_of_record) = 'Sunday' AND HOUR(time_of_record) = 9)
GROUP BY
Time_Period;
```


#### 11.Query to find out how many employees live in Dahabu (Use this for Kilimani and Harare,
change as needed)
```sql
SELECT COUNT(*) AS number_of_employees
FROM employee
WHERE town_name = 'Dahabu';
```

#### 12. Query for worst performing employees:
```sql
SELECT 
    assigned_employee_id AS Employee_ID,
    COUNT(visit_count) AS Number_of_Visits
FROM
    visits
GROUP BY assigned_employee_id
ORDER BY Number_of_Visits ASC
LIMIT 3;
```




#### 13. Query find out how many sites the two worst-performing employees (Employee IDs 20 and 22) visited
```sql
SELECT 
    assigned_employee_id AS Employee_ID,
    COUNT(visit_count) AS Number_of_Visits
FROM
    visits
WHERE
    assigned_employee_id IN (20 , 22)
GROUP BY Assigned_employee_id;
```


Water Accessibility and infrastructure summary report
This survey aimed to identify the water sources people use and determine both the total and average number of users for each source.
Additionally, it examined the duration citizens typically spend in queues to access water.




Insights
1. Most water sources are rural.
2. 43% of our people are using shared taps. 2000 people often share one tap.
3. 31% of our population has water infrastructure in their homes, but within that group, 45% face non-functional systems due to issues with pipes,
pumps, and reservoirs.
4. 18% of our people are using wells of which, but within that, only 28% are clean..
5. Our citizens often face long wait times for water, averaging more than 120 minutes.
6. In terms of queues:
- Queues are very long on Saturdays.
- Queues are longer in the mornings and evenings.
- Wednesdays and Sundays have the shortest queues.




Start of our plan
We have started thinking about a plan:
1. We want to focus our efforts on improving the water sources that affect the most people.
- Most people will benefit if we improve the shared taps first.
- Wells are a good source of water, but many are contaminated. Fixing this will benefit a lot of people.
- Fixing existing infrastructure will help many people. If they have running water again, they won't have to queue, thereby shorting queue times for
others. So we can solve two problems at once.
- Installing taps in homes will stretch our resources too thin, so for now, if the queue times are low, we won't improve that source.
2. Most water sources are in rural areas. We need to ensure our teams know this as this means they will have to make these repairs/upgrades in
rural areas where road conditions, supplies, and labour are harder challenges to overcome.




Practical solutions
1. If communities are using rivers, we can dispatch trucks to those regions to provide water temporarily in the short term, while we send out
crews to drill for wells, providing a more permanent solution.
2. If communities are using wells, we can install filters to purify the water. For wells with biological contamination, we can install UV filters that
kill microorganisms, and for *polluted wells*, we can install reverse osmosis filters. In the long term, we need to figure out why these sources
are polluted.
3. For shared taps, in the short term, we can send additional water tankers to the busiest taps, on the busiest days. We can use the queue time
pivot table we made to send tankers at the busiest times. Meanwhile, we can start the work on installing extra taps where they are needed.
According to UN standards, the maximum acceptable wait time for water is 30 minutes. With this in mind, our aim is to install taps to get
queue times below 30 min.
4. Shared taps with short queue times (< 30 min) represent a logistical challenge to further reduce waiting times. The most effective solution,
installing taps in homes, is resource-intensive and better suited as a long-term goal.
5. Addressing broken infrastructure offers a significant impact even with just a single intervention. It is expensive to fix, but so many people
can benefit from repairing one facility. For example, fixing a reservoir or pipe that multiple taps are connected to. We will have to find the
commonly affected areas though to see where the problem actually is.

Think these through a bit and in the meantime I'll send out some emails to get estimates of the cost to repair or improve each of these sources.


## Weaving the data threads of Maji Ndogo's narrative  (Level 3)

#### looking at the relationship between the visits and water_quality tables. For every entry in the visits table, there should be one unique corresponding record in the water_quality table. This means each visit recorded is associated with a specific water quality score, ensuring a one-to-one relationship between the visits and water_quality tables. But if we look at the ERD, it shows a many-to-one relationship. This does not agree with our thinking. Errors like these can cause problems, so let's fix that by changing it to one-to-one relationship.



### Integrating the Auditor's report

#### A .csv file of the auditor's results. You should get it loaded into SQL. You will have to think back to the start of our journey on how to do that


```sql
DROP TABLE IF EXISTS auditor_report;
CREATE TABLE auditor_report (
location_id VARCHAR(32),
type_of_water_source VARCHAR(64),
true_water_source_score int DEFAULT NULL,
statements VARCHAR(255)
);
```
#### Wow! First off, it looks like we have 1620 records, or sites that they re-visited. I see a location_id, type of water source at that location, and the quality score of the water source, that is now independently measured. Our auditor also investigated each site a bit by speaking to a few locals. Their statements are also captured in his results.

#### We need to tackle a couple of questions here.
#### 1. Is there a difference in the scores?
#### 2. If so, are there patterns?

#### For the first question, we will have to compare the quality scores in the water_quality table to the auditor's scores. The auditor_report table used location_id, but the quality scores table only has a record_id we can use. The visits table links location_id and record_id, so we can link the auditor_report table and water_quality using the visits table. So first, grab the location_id and true_water_source_score columns from auditor_report.Now, we join the visits table to the auditor_report table. Make sure to grab subjective_quality_score, record_id and location_id.

```sql
SELECT
auditor_report.location_id AS audit_location,
auditor_report.true_water_source_score,
visits.location_id AS visit_location,
visits.record_id
FROM
auditor_report
JOIN
visits
ON auditor_report.location_id = visits.location_id;
```



Now that we have the record_id for each location, our next step is to retrieve the corresponding scores from the water_quality table. We
are particularly interested in the subjective_quality_score. To do this, we'll JOIN the visits table and the water_quality table, using the
record_id as the connecting key.

```sql
SELECT 
    auditor_report.location_id,
    auditor_report.true_water_source_score AS Auditor_Score,
    visits.record_id,
    water_quality.subjective_quality_score AS Employee_Score
FROM
    auditor_report 
        JOIN
    visits  ON auditor_report.location_id = visits.location_id
        JOIN
    water_quality  ON visits.record_id = water_quality.record_id
```

#### OR

```sql
SELECT 
    ar.location_id,
    ar.true_water_source_score AS Auditor_Score,
    v.record_id,
    wq.subjective_quality_score AS Employee_Score
FROM
    auditor_report ar
        JOIN
    visits v ON ar.location_id = v.location_id
        JOIN
    water_quality wq ON v.record_id = wq.record_id;
```



#### Ok, let's analyse! A good starting point is to check if the auditor's and exployees' scores agree. There are many ways to do it. We can have a WHERE clause and check if surveyor_score = auditor_score, or we can subtract the two scores and check if the result is 0


```sql
SELECT 
    ar.location_id,
    ar.true_water_source_score AS Auditor_Score,
    v.record_id,
    wq.subjective_quality_score AS Employee_Score
FROM
    auditor_report ar
        JOIN
    visits v ON ar.location_id = v.location_id
        JOIN
    water_quality wq ON v.record_id = wq.record_id
    Where wq.subjective_quality_score = ar.true_water_source_score;
```


#### Some of the locations were visited multiple times, so these records are duplicated here. To fix it, we set visits.visit_count = 1 in the WHERE clause. Make sure you reference the alias you used for visits in the join.

```sql
SELECT 
    ar.location_id,
    ar.true_water_source_score AS Auditor_Score,
    v.record_id,
    wq.subjective_quality_score AS Employee_Score
FROM
    auditor_report ar
        JOIN
    visits v ON ar.location_id = v.location_id
        JOIN
    water_quality wq ON v.record_id = wq.record_id
WHERE
    wq.subjective_quality_score = ar.true_water_source_score
        AND v.visit_count = 1;
```

#### With the duplicates removed I now get 1518 rows. What does this mean considering the auditor visited 1620 sites? I think that is an excellent result as 1518/1620 = 94% of the records the auditor checked were correct!! But that means that 102 records are incorrect. So let's look at the incorrect ones  by adding one character in the last query! THE ERROR THAT RETURN 102 INCORRECT RECORDS 

```sql        
SELECT 
    ar.location_id,
    ar.true_water_source_score AS Auditor_Score,
    v.record_id,
    wq.subjective_quality_score AS Employee_Score
FROM
    auditor_report ar
        JOIN
    visits v ON ar.location_id = v.location_id
        JOIN
    water_quality wq ON v.record_id = wq.record_id
WHERE
    wq.subjective_quality_score != ar.true_water_source_score    --------INCORRECT RECORD 
        AND v.visit_count = 1;
```


#### Since we used some of this data in our previous analyses, we need to make sure those results are still valid, now we know some of them are incorrect. We didn't use the scores that much, but we relied a lot on the type_of_water_source, so let's check if there are any errors there. So, to do this, we need to grab the type_of_water_source column from the water_source table and call it survey_source, using the source_id column to JOIN. Also select the type_of_water_source from the auditor_report table, and call it auditor_source.


    
```sql     
SELECT 
    v.record_id AS record_id,
    a.location_id AS location_id,
    w.subjective_quality_score AS employee_score,
    a.true_water_source_score AS auditor_score,
    ws.type_of_water_source AS survey_source,
    a.type_of_water_source AS auditor_source
FROM
    visits AS v
        JOIN
    water_source AS ws ON v.source_id = ws.source_id
        JOIN
    auditor_report AS a ON a.location_id = v.location_id
        JOIN
    water_quality AS w ON v.record_id = w.record_id
WHERE
    (a.true_water_source_score = w.subjective_quality_score
        OR v.visit_count = 1)
        AND a.true_water_source_score <> w.subjective_quality_score;
```


#### we noticed  that the types of sources look the same! So even though the scores are wrong, the integrity of the type_of_water_source data we analysed last time is not affected. Once you're done, remove the columns and JOIN statement for water_sources again

```sql 
SELECT 
    v.record_id AS record_id,
    a.location_id AS location_id,
    w.subjective_quality_score AS employee_score,
    a.true_water_source_score AS auditor_score,
    a.type_of_water_source AS auditor_source
FROM
    visits AS v
        JOIN
    auditor_report AS a ON a.location_id = v.location_id
        JOIN
    water_quality AS w ON v.record_id = w.record_id
WHERE
    (a.true_water_source_score = w.subjective_quality_score
        OR v.visit_count = 1)
        AND a.true_water_source_score <> w.subjective_quality_score;
```


### Linking records to employees
#### Next up, let's look at where these errors may have come from. At some of the locations, employees assigned scores incorrectly, and those records ended up in this results set.

#### I think there are two reasons this can happen.
#### 1. These workers are all humans and make mistakes so this is expected.
#### 2. Unfortunately, the alternative is that someone assigned scores incorrectly on purpose!

#### In either case, the employees are the source of the errors, so let's JOIN the assigned_employee_id for all the people on our list from the visits table to our query. Remember, our query shows the shows the 102 incorrect records, so when we join the employee data, we can see which employees made these incorrect records.

```sql 
SELECT 
    v.record_id AS record_id,
    a.location_id AS location_id,
    w.subjective_quality_score AS employee_score,
    a.true_water_source_score AS auditor_score,
    a.type_of_water_source AS auditor_source
    v.assigned_employee_id AS Employee_id
FROM
    visits AS v
        JOIN
    auditor_report AS a ON a.location_id = v.location_id
        JOIN
    water_quality AS w ON v.record_id = w.record_id
        JOIN
    employee AS e ON e.assigned_employee_id = v.assigned_employee_id

WHERE
    (a.true_water_source_score = w.subjective_quality_score
        OR v.visit_count = 1)
        AND a.true_water_source_score <> w.subjective_quality_score;
```


#### So now we can link the incorrect records to the employees who recorded them. The ID's don't help us to identify them. We have employees' names stored along with their IDs, so let's fetch their names from the employees table instead of the ID's

```sql
SELECT 
    v.record_id AS record_id,
    a.location_id AS location_id,
    w.subjective_quality_score AS employee_score,
    a.true_water_source_score AS auditor_score,
    a.type_of_water_source AS auditor_source,
    e.employee_name AS Employee_name
FROM
    visits AS v
        JOIN
    auditor_report AS a ON a.location_id = v.location_id
        JOIN
    water_quality AS w ON v.record_id = w.record_id
        JOIN
    employee AS e ON e.assigned_employee_id = v.assigned_employee_id
WHERE
    (a.true_water_source_score = w.subjective_quality_score
        OR v.visit_count = 1)
        AND a.true_water_source_score <> w.subjective_quality_score;
```



#### Well this query is massive and complex, so maybe it is a good idea to save this as a CTE, so when we do more analysis, we can just call that CTE like it was a table. Call it something like Incorrect_records. Once you are done, check if this query SELECT * FROM Incorrect_records, gets the same table back. Now that we defined Incorrect_records, we can query it like any other table.

```sql
WITH
 Incorrect_records AS (
SELECT
  ar.location_id,
  ar.true_water_source_score AS Auditor_Score,
  v.record_id,
  em.employee_name,
  wq.subjective_quality_score AS Employee_score,
  v.visit_count
FROM
  auditor_report AS ar
JOIN visits AS v
  ON ar.location_id = v.location_id
JOIN water_quality AS wq
  ON v.record_id = wq.record_id
JOIN employee AS em
	ON em.assigned_employee_id = v.assigned_employee_id
  WHERE wq.subjective_quality_score !=  ar.true_water_source_score
  AND  v.visit_count = 1 )
SELECT *
	
FROM 
	Incorrect_records;
```



#### Let's first get a unique list of employees from this table. Think back to the start of your SQL journey to answer this one. I got 17 employees.

```sql
WITH
 Incorrect_records AS (
SELECT
  ar.location_id,
  ar.true_water_source_score AS Auditor_Score,
  v.record_id,
  em.employee_name,
  wq.subjective_quality_score AS Employee_score,
  v.visit_count
FROM
  auditor_report AS ar
JOIN visits AS v
  ON ar.location_id = v.location_id
JOIN water_quality AS wq
  ON v.record_id = wq.record_id
JOIN employee AS em
	ON em.assigned_employee_id = v.assigned_employee_id
  WHERE wq.subjective_quality_score !=  ar.true_water_source_score
  AND  v.visit_count = 1 )
SELECT 
	DISTINCT employee_name
FROM 
	Incorrect_records;
```



#### Next, let's try to calculate how many mistakes each employee made. So basically we want to count how many times their name is in Incorrect_records list, and then group them by name, right?

```sql
WITH
 Incorrect_records AS (
SELECT
  ar.location_id,
  ar.true_water_source_score AS Auditor_Score,
  v.record_id,
  em.employee_name,
  wq.subjective_quality_score AS Employee_score,
  v.visit_count
FROM
  auditor_report AS ar
JOIN visits AS v
  ON ar.location_id = v.location_id
JOIN water_quality AS wq
  ON v.record_id = wq.record_id
JOIN employee AS em
	ON em.assigned_employee_id = v.assigned_employee_id
  WHERE wq.subjective_quality_score !=  ar.true_water_source_score
  AND  v.visit_count = 1 )
SELECT 
	DISTINCT employee_name,
    COUNT(employee_name) AS number_of_mistakes
FROM 
	Incorrect_records
GROUP BY employee_name
ORDER BY number_of_mistakes DESC;
```


#### It looks like some of our surveyors are making a lot of "mistakes" while many of the other surveyors are only making a few. we don't like where this is going

## Gathering some evidence
Ok, so thinking about this a bit. How would we go about finding out if any of our employees are corrupt?

#### Let's say all employees make mistakes, if someone is corrupt, they will be making a lot of "mistakes", more than average, for example. But someone could just be clumsy, so we should try to get more evidence... Our auditor did say some of the things he heard on the streets were quite shady, and he recorded this in the statements column. Considering both of these sources should give us a pretty reliable answer. So let's try to find all of the employees who have an above-average number of mistakes. Let's break it down into steps first:
#### 1. We have to first calculate the number of times someone's name comes up. (we just did that in the previous query). Let's call it error_count.
#### 2. Then, we need to calculate the average number of mistakes employees made. We can do that by taking the average of the previous query's results. Something like this : 

```sql
SELECT
AVG(number_of_mistakes)
FROM
error_count;
```

#### Let's call that result avg_error_count_per_empl, which would be a scalar value.

```sql
WITH Incorrect_records AS (
SELECT
a.location_id AS location_id,
v.record_id AS record_id,
a.true_water_source_score AS auditor_score,
w.subjective_quality_score AS employee_score,
e.employee_name
FROM
auditor_report AS a
JOIN
visits AS v
ON a.location_id = v.location_id
LEFT JOIN
water_quality AS w
ON v.record_id = w.record_id
LEFT JOIN
employee AS e
ON e.assigned_employee_id = v.assigned_employee_id
WHERE
(a.true_water_source_score = w.subjective_quality_score OR v.visit_count = 1)
AND a.true_water_source_score <> w.subjective_quality_score
),
ErrorCount AS (
SELECT
employee_name,
COUNT(*) AS number_of_mistakes
FROM Incorrect_records
GROUP BY employee_name
)
SELECT
AVG(number_of_mistakes) AS avg_error_count_per_empl
FROM ErrorCount;
```



#### 3. Finaly we have to compare each employee's error_count with avg_error_count_per_empl. We will call this results set our suspect_list. Remember that we can't use an aggregate result in WHERE, so we have to use avg_error_count_per_empl as a subquery.

```sql
SELECT
employee_name,
number_of_mistakes
FROM
error_count
WHERE
number_of_mistakes > (avg_error_count_per_empl);
```



## QUERY suspect list

```sql
WITH Incorrect_records AS (
SELECT
a.location_id AS location_id,
v.record_id AS record_id,
a.true_water_source_score AS auditor_score,
w.subjective_quality_score AS employee_score,
e.employee_name
FROM
auditor_report AS a
JOIN
visits AS v
ON a.location_id = v.location_id
LEFT JOIN
water_quality AS w
ON v.record_id = w.record_id
LEFT JOIN
employee AS e
ON e.assigned_employee_id = v.assigned_employee_id
WHERE
(a.true_water_source_score = w.subjective_quality_score OR v.visit_count = 1)
AND a.true_water_source_score <> w.subjective_quality_score
),
ErrorCount AS (
SELECT
employee_name,
COUNT(*) AS number_of_mistakes
FROM Incorrect_records
GROUP BY employee_name
),
SuspectList AS (
SELECT
employee_name,
number_of_mistakes
FROM
ErrorCount
WHERE
number_of_mistakes > (SELECT AVG(number_of_mistakes) FROM ErrorCount)
)
SELECT
employee_name,
number_of_mistakes
FROM SuspectList;
```

#### 1. Let's start by cleaning up our code a bit. First, Incorrect_records is a result we'll be using for the rest of the analysis, but it makes the query a bit less readable. So, let's convert it to a VIEW. We can then use it as if it was a table. It will make our code much simpler to read, but, it comes at a cost. We can add comments to CTEs in our code, so if we return to that query a year later, we can read those comments and quickly understand what Incorrect_records represents. If we save it as a VIEW, it is not as obvious. So we should add comments in places where we use Incorrect_records.

#### So, replace WITH with CREATE VIEW like this, and note that I added the statements column to this table in line 8 too:

```sql
CREATE VIEW
 Incorrect_records AS (
SELECT
  ar.location_id,
  ar.true_water_source_score AS Auditor_Score,
  v.record_id,
  em.employee_name,
  wq.subjective_quality_score AS Employee_score,
  ar.statements AS statements,
  v.visit_count
FROM
  auditor_report AS ar
JOIN visits AS v
  ON ar.location_id = v.location_id
JOIN water_quality AS wq
  ON v.record_id = wq.record_id
JOIN employee AS em
	ON em.assigned_employee_id = v.assigned_employee_id
  WHERE wq.subjective_quality_score !=  ar.true_water_source_score
  AND  v.visit_count = 1);
```
 

#### Now, calling SELECT * FROM Incorrect_records gives us the same result as the CTE did.

```sql
SELECT * FROM Incorrect_records;
```

#### Next, we convert the query error_count, we made earlier, into a CTE. Test it to make sure it gives the same result again, using SELECT * FROM Incorrect_records. On large queries like this, it is better to build the query, and test each step, because fixing errors becomes harder as the query grows.

#### 2. Now calculate the average of the number_of_mistakes in error_count. You should get a single value.

#### 3. To find the employees who made more mistakes than the average person, we need the employee's names, the number of mistakes each one made, and filter the employees with an above-average number of mistakes.
#### HINT: Use SELECT AVG(mistake_count) FROM error_count as a custom filter in the WHERE part of our query.

```sql
WITH
 error_count AS (
SELECT 
	DISTINCT employee_name,
    COUNT(employee_name) AS number_of_mistakes
FROM 
	Incorrect_records
GROUP BY employee_name
ORDER BY number_of_mistakes DESC)
SELECT 
		*
	FROM error_count
	WHERE 
		number_of_mistakes > (SELECT AVG(number_of_mistakes) FROM error_count);
```
#### These are the employees who made more mistakes, on average, than their peers, so let's have a closer look at them. We should look at the Incorrect_records table again, and isolate all of the records these four employees gathered. We should also look at the statements for these records to look for patterns. First, convert the suspect_list to a CTE, so we can use it to filter the records from these four employees. Make sure you get the names of the four "suspects", and their mistake count as a result, using SELECT employee_name FROM suspect_list.

#### You should get a column of names back. So let's just recap here...
#### 1. We use Incorrect_records to find all of the records where the auditor and employee scores don't match.
#### 2. We then used error_count to aggregate the data, and got the number of mistakes each employee made.
#### 3. Finally, suspect_list retrieves the data of employees who make an above-average number of mistakes.
#### Now we can filter that Incorrect_records CTE to identify all of the records associated with the four employees we identified.

####  Firstly, let's add the statements column to the Incorrect_records CTE. Then pull up all of the records where the employee_name is in the suspect list. HINT: Use SELECT employee_name FROM suspect_list as a subquery in WHERE.



### QUERY suspect list

```sql
WITH Incorrect_records AS (
SELECT
a.location_id AS location_id,
v.record_id AS record_id,
a.true_water_source_score AS auditor_score,
w.subjective_quality_score AS employee_score,
e.employee_name
FROM
auditor_report AS a
JOIN
visits AS v
ON a.location_id = v.location_id
LEFT JOIN
water_quality AS w
ON v.record_id = w.record_id
LEFT JOIN
employee AS e
ON e.assigned_employee_id = v.assigned_employee_id
WHERE
(a.true_water_source_score = w.subjective_quality_score OR v.visit_count = 1)
AND a.true_water_source_score <> w.subjective_quality_score
),
ErrorCount AS (
SELECT
employee_name,
COUNT(*) AS number_of_mistakes
FROM Incorrect_records
GROUP BY employee_name
),
SuspectList AS (
SELECT
employee_name,
number_of_mistakes
FROM
ErrorCount
WHERE
number_of_mistakes > (SELECT AVG(number_of_mistakes) FROM ErrorCount)
)
SELECT
employee_name,
number_of_mistakes
FROM SuspectList;
```



### QUERY to add the statements column to the Incorrect_records CTE. Then pull up all of the records where the employee_name is in the suspect list
```sql
WITH Incorrect_records AS (
SELECT
a.location_id AS location_id,
v.record_id AS record_id,
a.true_water_source_score AS auditor_score,
w.subjective_quality_score AS employee_score,
a.statements, -- Add statements column
e.employee_name
FROM
auditor_report AS a
JOIN
visits AS v
ON a.location_id = v.location_id
LEFT JOIN
water_quality AS w
ON v.record_id = w.record_id
LEFT JOIN
employee AS e
ON e.assigned_employee_id = v.assigned_employee_id
WHERE
(a.true_water_source_score = w.subjective_quality_score OR v.visit_count = 1)
AND a.true_water_source_score <> w.subjective_quality_score
),
ErrorCount AS (
SELECT
employee_name,
COUNT(*) AS number_of_mistakes
FROM Incorrect_records
GROUP BY employee_name
),
SuspectList AS (
SELECT
employee_name,
number_of_mistakes
FROM
ErrorCount
WHERE
number_of_mistakes > (SELECT AVG(number_of_mistakes) FROM ErrorCount)
)
SELECT
i.employee_name,
i.location_id,
i.statements
FROM
Incorrect_records AS i
WHERE
i.employee_name IN (SELECT employee_name FROM SuspectList);
```

#### If you have a look, you will notice some alarming statements about these four officials (look at these records: AkRu04508, AkRu07310, KiRu29639, AmAm09607, for example. See how the word "cash" is used a lot in these statements.

#### Filter the records that refer to "cash". Let's just do one more check to make sure... 
#### Check if there are any employees in the Incorrect_records table with statements mentioning "cash" that are not in our suspect list. This should be as simple as adding one word.
#### I get an empty result, so no one, except the four suspects, has these allegations of bribery.



### QUERY to filter the word cash

```sql
WITH Incorrect_records AS (
SELECT
a.location_id AS location_id,
v.record_id AS record_id,
a.true_water_source_score AS auditor_score,
w.subjective_quality_score AS employee_score,
a.statements,
e.employee_name
FROM
auditor_report AS a
JOIN
visits AS v
ON a.location_id = v.location_id
LEFT JOIN
water_quality AS w
ON v.record_id = w.record_id
LEFT JOIN
employee AS e
ON e.assigned_employee_id = v.assigned_employee_id
WHERE
(a.true_water_source_score = w.subjective_quality_score OR v.visit_count = 1)
AND a.true_water_source_score <> w.subjective_quality_score
AND a.statements LIKE '%cash%' -- Filter for "cash" in statements
),
ErrorCount AS (
SELECT
employee_name,
COUNT(*) AS number_of_mistakes
FROM Incorrect_records
GROUP BY employee_name
),
SuspectList AS (
SELECT
employee_name,
number_of_mistakes
FROM
ErrorCount
WHERE
number_of_mistakes > (SELECT AVG(number_of_mistakes) FROM ErrorCount)
)
SELECT
i.employee_name,
i.location_id,
i.statements
FROM
Incorrect_records AS i
WHERE
i.employee_name IN (SELECT employee_name FROM SuspectList);
```

#### OR 

```sql
WITH Incorrect_records AS (
SELECT
a.location_id AS location_id,
v.record_id AS record_id,
a.true_water_source_score AS auditor_score,
w.subjective_quality_score AS employee_score,
a.statements,
e.employee_name
FROM
auditor_report AS a
JOIN
visits AS v
ON a.location_id = v.location_id
LEFT JOIN
water_quality AS w
ON v.record_id = w.record_id
LEFT JOIN
employee AS e
ON e.assigned_employee_id = v.assigned_employee_id
WHERE
(a.true_water_source_score = w.subjective_quality_score OR v.visit_count = 1)
AND a.true_water_source_score <> w.subjective_quality_score
),
ErrorCount AS (
SELECT
employee_name,
COUNT(*) AS number_of_mistakes
FROM Incorrect_records
GROUP BY employee_name
),
SuspectList AS (
SELECT
employee_name,
number_of_mistakes
FROM
ErrorCount
WHERE
number_of_mistakes > (SELECT AVG(number_of_mistakes) FROM ErrorCount)
)
SELECT
i.employee_name,
i.location_id,
i.statements
FROM
Incorrect_records AS i
WHERE
i.employee_name IN (SELECT employee_name FROM SuspectList)
AND i.location_id IN ('AkRu04508', 'AkRu07310', 'KiRu29639', 'AmAm09607')
AND i.statements LIKE '%CASH%';
```






## FINAL QUERY 
```sql
WITH Incorrect_records AS (
SELECT
a.location_id AS audit_location,
a.true_water_source_score as auditor_score,
v.record_id,
wq.subjective_quality_score as surveyor_score,
v.assigned_employee_id,
e.employee_name,
a.statements
FROM
auditor_report as a
JOIN
Visits as V
on a.location_id = v.location_id
JOIN
Water_quality as wq
ON wq.record_id = v.record_id
JOIN
employee as e
ON e.assigned_employee_id = v.assigned_employee_id
WHERE
wq.subjective_quality_score - a.true_water_source_score != 0
and v.visit_count = 1
),
Error_Count AS (
SELECT DISTINCT
employee_name,
COUNT(*) as number_of_mistakes
FROM
Incorrect_records
GROUP BY
employee_name
),
Avg_error_count_per_employee AS (
SELECT
AVG(number_of_mistakes) as avg_error_count_per_empl
FROM
error_count
),
Suspect_list AS (
SELECT
employee_name,
number_of_mistakes
FROM
error_count
WHERE
number_of_mistakes > (SELECT
AVG(number_of_mistakes) as
avg_error_count_per_empl
FROM
error_count)
)
SELECT
employee_name,
audit_location,
statements
FROM
incorrect_records
WHERE
employee_name NOT IN (SELECT employee_name FROM suspect_list)
AND statements LIKE '%CASH%';
```

#### So we can sum up the evidence we have for Zuriel Matembo, Malachi Mavuso, Bello Azibo and Lalitha Kaburi:
#### 1. They all made more mistakes than their peers on average.
#### 2. They all have incriminating statements made against them, and only them. Keep in mind, that this is not decisive proof, but it is concerning enough that we should flag it. Pres. Naledi has worked hard to stamp out corruption, so she would urge us to report this. I am a bit shocked to be honest! After all our teams set out to do, it is hard for me to uncover this. I'll let Pres. Naledi know what we found out














    
