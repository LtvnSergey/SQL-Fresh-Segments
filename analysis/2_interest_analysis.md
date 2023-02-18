# Project: Fresh Segments

# Interest Analysis



--- 
### 1. Which interests have been present in all `month_year`?

````sql
WITH all_dates_interests AS (
SELECT
		DISTINCT(interest_id::INT)
FROM
		interest_metrics
GROUP BY
		interest_id
HAVING
		COUNT(DISTINCT month_year) =
		(
	SELECT
			COUNT(DISTINCT (month_year)) AS dist_dates_count
	FROM
			interest_metrics )
)
	

SELECT
	interest_name
FROM
	interest_map
WHERE
	id IN (
	SELECT
		*
	FROM
		all_dates_interests)
````

|interest_name|
|-------------|
|Luxury Retail Researchers|
|Brides & Wedding Planners|
|Vacation Planners|
|Thrift Store Shoppers|
|NBA Fans|


````sql

SELECT
	COUNT(interest_name)
FROM
	interest_map
WHERE
	id IN (
	SELECT
		*
	FROM
		all_dates_interests)
````

|count|
|-----|
|480|

- 480 interests are present during all `month_year` dates

  
### 2. Using this same `total_months` measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?

````sql 
WITH month_count AS (
SELECT
	DISTINCT(interest_id),
	COUNT(month_year) AS month_count
FROM
	interest_metrics
GROUP BY
	interest_id
),

interests_count AS (
SELECT
	month_count,
	COUNT(interest_id) AS interest_count
FROM
	month_count
GROUP BY
	month_count
),

cumulative_percentage AS (
SELECT 
	*, 
	ROUND(SUM(interest_count) OVER (ORDER BY month_count DESC) * 100 / 
	(SELECT SUM(interest_count) FROM interests_count), 2) AS cumulative_percent
FROM
	interests_count
GROUP BY
	month_count,
	interest_count
)

SELECT
	*
FROM
	cumulative_percentage
WHERE
	cumulative_percent > 90
````

|month_count|interest_count|cumulative_percent|
|-----------|--------------|------------------|
|6|33|90.85|
|5|38|94.01|
|4|32|96.67|
|3|15|97.92|
|2|12|98.92|
|1|13|100.00|










