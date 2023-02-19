# Project: Fresh Segments

# Index Analysis


- The `index_value` is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

- Average `composition` can be calculated by dividing the `composition` column by the `index_value` column rounded to 2 decimal places.


### 1. What is the top 10 interests by the average composition for each month?

````sql
 
WITH 
composition_ranking AS (
	SELECT 
		month_year,
		interest_id::INTEGER,
		ROUND((composition/index_value)::NUMERIC, 2) AS avg_composition,
		RANK() OVER (PARTITION BY month_year ORDER BY ROUND((composition/index_value)::NUMERIC, 2) DESC) AS composition_rank
	FROM
		interest_metrics im
)

SELECT 
	cm.month_year,
	im.interest_name,
	cm.composition_rank,
	cm.avg_composition
FROM 
	composition_ranking cm
LEFT JOIN
	interest_map im
ON cm.interest_id = im.id 
WHERE cm.composition_rank <= 10
ORDER BY cm.month_year, cm.composition_rank
LIMIT 10
````

|month_year|interest_name|composition_rank|avg_composition|
|----------|-------------|----------------|---------------|
|2018-07-01|Las Vegas Trip Planners|1|7.36|
|2018-07-01|Gym Equipment Owners|2|6.94|
|2018-07-01|Cosmetics and Beauty Shoppers|3|6.78|
|2018-07-01|Luxury Retail Shoppers|4|6.61|
|2018-07-01|Furniture Shoppers|5|6.51|
|2018-07-01|Asian Food Enthusiasts|6|6.10|
|2018-07-01|Recently Retired Individuals|7|5.72|
|2018-07-01|Family Adventures Travelers|8|4.85|
|2018-07-01|Work Comes First Travelers|9|4.80|
|2018-07-01|HDTV Researchers|10|4.71|

--- 


### 2. For all of these top 10 interests - which interest appears the most often?

````sql
 WITH 
composition_ranking AS (
  SELECT 
      month_year,
      interest_id::INTEGER,
      ROUND((composition / index_value)::NUMERIC, 2) AS avg_composition,
      RANK() OVER (PARTITION BY month_year
  ORDER BY
    ROUND((composition / index_value)::NUMERIC, 2) DESC) AS composition_rank
  FROM
      interest_metrics im
)

SELECT 
	DISTINCT(im.interest_name),
	COUNT(*) OVER (PARTITION BY interest_name) AS count_times
FROM 
	composition_ranking cm
LEFT JOIN
	interest_map im
ON
	cm.interest_id = im.id
WHERE
	cm.composition_rank <= 10
ORDER BY
	count_times DESC
LIMIT 5
````

|interest_name|count_times|
|-------------|-----------|
|Alabama Trip Planners|10|
|Luxury Bedding Shoppers|10|
|Solar Energy Researchers|10|
|New Years Eve Party Ticket Purchasers|9|
|Readers of Honduran Content|9|


### 3. 
