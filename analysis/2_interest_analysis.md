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

---
  
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

---

### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?


````sql
WITH month_count AS (
SELECT
	interest_id,
	COUNT(DISTINCT month_year) AS month_count
FROM
	interest_metrics
GROUP BY
	interest_id
HAVING
	COUNT(DISTINCT month_year) < 6
)

SELECT
	COUNT(interest_id) AS interests_to_remove
FROM
	interest_metrics
WHERE
	interest_id IN (
	SELECT
		interest_id
	FROM
		month_count)
````		

|interests_to_remove|
|-------------------|
|400|

- We will need to remove 400 data points

---

### 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.

````sql
WITH month_count AS (
SELECT
	interest_id,
	COUNT(DISTINCT month_year) AS month_count
FROM
	interest_metrics
GROUP BY
	interest_id
HAVING
	COUNT(DISTINCT month_year) < 6
)


SELECT
	removed.month_year,
	present_interest,
	removed_interest,
	round(removed_interest * 100.0 /(removed_interest + present_interest), 2) AS percentage_removed
FROM
	(
	SELECT
		month_year,
		count(*) AS removed_interest
	FROM
		interest_metrics
	WHERE
		interest_id IN (
		SELECT
			interest_id
		FROM
			month_count)
	GROUP BY
		month_year
) removed
LEFT JOIN 

(
	SELECT
		month_year,
		count(*) AS present_interest
	FROM
		interest_metrics
	WHERE
		interest_id NOT IN (
		SELECT
			interest_id
		FROM
			month_count)
	GROUP BY
		month_year
) present
ON
	removed.month_year = present.month_year
ORDER BY
	removed.month_year
````

|month_year|present_interest|removed_interest|percentage_removed|
|----------|----------------|----------------|------------------|
|2018-07-01|709|20|2.74|
|2018-08-01|752|15|1.96|
|2018-09-01|774|6|0.77|
|2018-10-01|853|4|0.47|
|2018-11-01|925|3|0.32|
|2018-12-01|986|9|0.90|
|2019-01-01|966|7|0.72|
|2019-02-01|1072|49|4.37|
|2019-03-01|1078|58|5.11|
|2019-04-01|1035|64|5.82|
|2019-05-01|827|30|3.50|
|2019-06-01|804|20|2.43|
|2019-07-01|836|28|3.24|
|2019-08-01|1062|87|7.57|

- Its very important for segment analysis to have use as wide specrtum of points across data as possible, 
but since we are removing small percantage of data for each month (< 8 %), it shouldn't harm further analysis.

---

### 5. After removing these interests - how many unique interests are there for each month?

````sql
WITH month_count AS (
SELECT
	interest_id,
	COUNT(DISTINCT month_year) AS month_count
FROM
	interest_metrics
GROUP BY
	interest_id
HAVING
	COUNT(DISTINCT month_year) < 6
)


SELECT
	month_year,
	count(*) AS unique_interests
FROM
	interest_metrics
WHERE
	interest_id NOT IN (
	SELECT
		interest_id
	FROM
		month_count)
GROUP BY
	month_year
````

|month_year|unique_interests|
|----------|----------------|
|2018-10-01|853|
|2019-04-01|1035|
|2018-11-01|925|
|2018-12-01|986|
|2019-07-01|836|
|2019-05-01|827|
|2019-03-01|1078|
|2019-02-01|1072|
|2018-09-01|774|
|2019-06-01|804|
|2018-08-01|752|
|2019-01-01|966|
|2019-08-01|1062|
|2018-07-01|709|

- After removing interests that are less presented, there are approximatly 700 - 1080 interests left per each month
