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

### 2. Updating the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month


````sql
ALTER TABLE interest_metrics
    ALTER COLUMN month_year TYPE DATE USING to_date(CONCAT(_year,'-',_month,'-01'), 'YYYY-MM-DD');
````

---

### 3. Count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest)

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


### 5. Amount of `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table

````sql
SELECT 
	COUNT(DISTINCT interest_id) AS interest_ids_not_in_map_table
FROM
	interest_metrics
WHERE
	interest_id::INT NOT IN 
(
	SELECT
		DISTINCT(id)
	FROM
		interest_map)
````

|interest_ids_not_in_map_table|
|-----------------------------|
|0|


````sql
SELECT 
	COUNT(DISTINCT id) AS interest_ids_not_in_metric_table
FROM
	interest_map
WHERE
	id::CHARACTER  NOT IN 
(
	SELECT
		DISTINCT(interest_id)
	FROM
		interest_metrics)
````

|interest_ids_not_in_metric_table|
|--------------------------------|
|10|


--- 

### 6. Summary of `id` values in the 'fresh_segments.interest_map' by its total record count in this table

````sql
WITH interest_id_count AS (
SELECT 
	COUNT(*) AS records_per_id
FROM
	interest_metrics
GROUP BY
	interest_id
)

SELECT
	MIN(records_per_id) AS min_records_per_id,
	MAX(records_per_id) AS max_records_per_id,
	ROUND(AVG(records_per_id), 0) AS avg_records_per_id
FROM
	interest_id_count
````

|min_records_per_id|max_records_per_id|avg_records_per_id|
|------------------|------------------|------------------|
|1|14|11|


- There are minimum 1 record per interest id, maximum 14 and average value 11

--- 

### 7. Combined table `fresh_segments.interest_metrics` and `fresh_segments.interest_map`

````sql
SELECT
	interest_id, 
	interest_name, 
	interest_summary, 
	composition, 
	index_value, 
	ranking, 
	percentile_ranking,
	_month, 
	_year, 
	month_year,
	created_at, 
	last_modified
FROM
	interest_metrics im
LEFT JOIN
	interest_map m	
ON
	im.interest_id::INT = m.id
````


|interest_id|interest_name|interest_summary|composition|index_value|ranking|percentile_ranking|_month|_year|month_year|created_at|last_modified|
|-----------|-------------|----------------|-----------|-----------|-------|------------------|------|-----|----------|----------|-------------|
|169|Social Liberals|People reading left-leaning social issues websites.|6.1|2.72|130|82.17|7|2018|2018-07-01|2016-05-26 14:57:59.000|2018-05-23 11:30:13.000|
|101|Flower & Gift Basket Shoppers|Consumers shopping for flowers and gift baskets.|7.38|2.72|130|82.17|7|2018|2018-07-01|2016-05-26 14:57:59.000|2018-05-23 11:30:12.000|
|6087|Metal and Rock Music Fans|People researching news and viewing online content focused on metal and rock music.|5.41|2.71|132|81.89|7|2018|2018-07-01|2017-03-27 16:59:29.000|2017-12-27 12:31:08.000|
|104|Country Music Fans|People reading about country music and country music stars.|5.42|2.71|132|81.89|7|2018|2018-07-01|2016-05-26 14:57:59.000|2018-05-23 11:30:12.000|
|6284|Gym Equipment Owners|People researching and comparing fitness trends and techniques. These consumers are more likely to spend money on gym equipment for their homes.|18.82|2.71|132|81.89|7|2018|2018-07-01|2017-04-18 15:46:10.000|2017-12-27 12:31:08.000|
|7557|Tailgaters||8.17|2.71|132|81.89|7|2018|2018-07-01|2017-07-20 17:31:31.000|2019-01-16 09:11:30.000|
|37|Jewelry & Watch Shoppers|Consumers researching high-end watch and jewelry brands.|7.82|2.71|132|81.89|7|2018|2018-07-01|2016-05-26 14:57:59.000|2018-05-23 11:30:12.000|
|6084|Media Buying Professionals|Professionals reading advertising industry news and media buying trends.|6.97|2.69|137|81.21|7|2018|2018-07-01|2017-03-27 16:59:29.000|2018-05-23 11:30:12.000|
|8|Business News Readers|Readers of online business news content.|6.67|2.68|138|81.07|7|2018|2018-07-01|2016-05-26 14:57:59.000|2018-05-23 11:30:12.000|
|5969|Luxury Bedding Shoppers|Consumers shopping for luxury bedding.|11.15|2.68|138|81.07|7|2018|2018-07-01|2017-03-08 09:57:00.000|2018-05-23 11:30:12.000|


--- 

### 8. Amount of records in joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table

````sql
SELECT
	COUNT(*) AS records_with_month_year_before_creation
FROM
	interest_metrics im
LEFT JOIN
	interest_map m	
ON
	im.interest_id::INT = m.id
WHERE
	im.month_year < m.created_at
````

|records_with_month_year_before_creation|
|---------------------------------------|
|188|


- There are 188 records which `month_year` value is before `created_at'	 value

````sql
SELECT
	MAX(ABS(EXTRACT(YEAR FROM(month_year - created_at)))) AS max_year_diff,
	MAX(ABS(EXTRACT(MONTH FROM(month_year - created_at)))) AS max_month_diff,
	MAX(ABS(EXTRACT(DAY FROM(month_year - created_at)))) AS max_day_diff
FROM
	interest_metrics im
LEFT JOIN
	interest_map m	
ON
	im.interest_id::INT = m.id
WHERE
	im.month_year < m.created_at
````

|max_year_diff|max_month_diff|max_day_diff|
|-------------|--------------|------------|
|0|0|16|

- These records with `month_year` before `created_at` are valid, because the difference between them are only in days and column `month_year` is only correct for months and years
