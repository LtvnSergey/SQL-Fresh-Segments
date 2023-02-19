# Project: Fresh Segments

# Index Analysis


- The `index_value` is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

- Average `composition` can be calculated by dividing the `composition` column by the `index_value` column rounded to 2 decimal places.


### 1. Top 10 interests by the average composition for each month

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

### 2. Most frequent interests among those top 10

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

---

### 3. Average of the average composition for the top 10 interests for each month

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
	ROUND(AVG(cm.avg_composition),2) AS avg_composition_per_month
FROM 
	composition_ranking cm
LEFT JOIN
	interest_map im
ON cm.interest_id = im.id 
WHERE cm.composition_rank <= 10
GROUP BY cm.month_year
ORDER BY cm.month_year
````

|month_year|avg_composition_per_month|
|----------|-------------------------|
|2018-07-01|6.04|
|2018-08-01|5.95|
|2018-09-01|6.90|
|2018-10-01|7.07|
|2018-11-01|6.62|
|2018-12-01|6.65|
|2019-01-01|6.32|
|2019-02-01|6.58|
|2019-03-01|6.12|
|2019-04-01|5.75|
|2019-05-01|3.54|
|2019-06-01|2.43|
|2019-07-01|2.77|
|2019-08-01|2.63|

---

### 4. 3 month rolling average of the max average composition value from September 2018 to August 2019, including the previous top ranking interests

````sql
 
WITH 
composition_ranking AS (
	SELECT 
		month_year,
		interest_id::INTEGER,
		ROUND((composition/index_value)::NUMERIC, 2) AS max_index_composition,
		RANK() OVER (PARTITION BY month_year ORDER BY ROUND((composition/index_value)::NUMERIC, 2) DESC) AS composition_rank
	FROM
		interest_metrics im
	ORDER BY month_year 
)

SELECT 
	cm.month_year,
	im.interest_name,
	cm.max_index_composition,
	ROUND(AVG(cm.max_index_composition) 
		OVER (ORDER BY cm.month_year 
			  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW),2) AS "3_month_moving_avg",
	CONCAT(LAG(im.interest_name) 
		OVER (ORDER BY  month_year), ' : ',LAG(cm.max_index_composition) 
			  OVER (ORDER BY month_year)) as "1_month_ago",
	CONCAT(LAG(im.interest_name, 2) 
		OVER (ORDER BY  month_year), ' : ',LAG(cm.max_index_composition, 2) 
			  OVER (ORDER BY month_year)) as "2_month_ago"
FROM 
	composition_ranking cm
LEFT JOIN
	interest_map im
ON cm.interest_id = im.id 
WHERE cm.composition_rank = 1
ORDER BY cm.month_year
OFFSET 2
````

|month_year|interest_name|max_index_composition|3_month_moving_avg|1_month_ago|2_month_ago|
|----------|-------------|---------------------|------------------|-----------|-----------|
|2018-09-01|Work Comes First Travelers|8.26|7.61|Las Vegas Trip Planners : 7.21|Las Vegas Trip Planners : 7.36|
|2018-10-01|Work Comes First Travelers|9.14|8.20|Work Comes First Travelers : 8.26|Las Vegas Trip Planners : 7.21|
|2018-11-01|Work Comes First Travelers|8.28|8.56|Work Comes First Travelers : 9.14|Work Comes First Travelers : 8.26|
|2018-12-01|Work Comes First Travelers|8.31|8.58|Work Comes First Travelers : 8.28|Work Comes First Travelers : 9.14|
|2019-01-01|Work Comes First Travelers|7.66|8.08|Work Comes First Travelers : 8.31|Work Comes First Travelers : 8.28|
|2019-02-01|Work Comes First Travelers|7.66|7.88|Work Comes First Travelers : 7.66|Work Comes First Travelers : 8.31|
|2019-03-01|Alabama Trip Planners|6.54|7.29|Work Comes First Travelers : 7.66|Work Comes First Travelers : 7.66|
|2019-04-01|Solar Energy Researchers|6.28|6.83|Alabama Trip Planners : 6.54|Work Comes First Travelers : 7.66|
|2019-05-01|Readers of Honduran Content|4.41|5.74|Solar Energy Researchers : 6.28|Alabama Trip Planners : 6.54|
|2019-06-01|Las Vegas Trip Planners|2.77|4.49|Readers of Honduran Content : 4.41|Solar Energy Researchers : 6.28|
|2019-07-01|Las Vegas Trip Planners|2.82|3.33|Las Vegas Trip Planners : 2.77|Readers of Honduran Content : 4.41|
|2019-08-01|Cosmetics and Beauty Shoppers|2.73|2.77|Las Vegas Trip Planners : 2.82|Las Vegas Trip Planners : 2.77|

---

### 5. Overall reasons why the max average composition might change from month to month

- Max average composition might change from month-to-month due to such reasons as:
	a. Some interest could be seasonal
	b. Customers become tired of similar interests (like 'Work Comes First Travalers)
	c. During the year an audience group in general could change
	d. Current bussiness model is not appropriate and thus it should be modified
