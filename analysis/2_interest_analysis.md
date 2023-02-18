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

  








