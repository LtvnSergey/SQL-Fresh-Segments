# Project: Fresh Segments

# Data Exploration and Cleansing

--- 

### 1. Missing values

````sql
SELECT 100-100*COUNT(_year)/COUNT(*) AS percent_missing FROM interest_metrics
````

|percent_missing|
|-----------|
|9|


- 9% of data in `interest_metrics` table have missing date values

- Lets delete rows with missing data, since we cannot use them in analysis

````sql
DELETE FROM interest_metrics WHERE month_year IS NULL;
````

--- 

### 2. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month


````sql
ALTER TABLE interest_metrics
    ALTER COLUMN month_year TYPE DATE USING to_date(CONCAT(_year,'-',_month,'-01'), 'YYYY-MM-DD');
````


### 3. What is count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest)?

````sql
SELECT month_year, COUNT(*) AS records
FROM interest_metrics 
GROUP BY month_year 
ORDER BY month_year 
````

|month_year|records|
|----------|-------|
|2018-07-01|729|
|2018-08-01|767|
|2018-09-01|780|
|2018-10-01|857|
|2018-11-01|928|
|2018-12-01|995|
|2019-01-01|973|
|2019-02-01|1121|
|2019-03-01|1136|
|2019-04-01|1099|
|2019-05-01|857|
|2019-06-01|824|
|2019-07-01|864|
|2019-08-01|1149|



--- 

### 5. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?


--- 

### 6. Summarise the id values in the fresh_segments.interest_map by its total record count in this table


--- 

### 7. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.


--- 

### 8. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
