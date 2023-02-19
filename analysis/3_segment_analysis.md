# Project: Fresh Segments

# Segment Analysis


--- 

### 1. Create table for segment analysis using filtered dataset by removing the interests with less than 6 months worth of data.

- Create `segment_analysis` table

````sql
CREATE TABLE fresh_segments.segment_analysis (
	interest_id INTEGER, 
	interest_name TEXT, 
	interest_summary TEXT, 
	composition FLOAT, 
	index_value FLOAT, 
	ranking INTEGER, 
	percentile_ranking FLOAT,
	month_year TIMESTAMP,
	created_at TIMESTAMP, 
	last_modified TIMESTAMP
)
````

- Insert values from joined tables `interest_metrics` and `interest_map` with filter

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


INSERT
	INTO
	fresh_segments.segment_analysis
SELECT 
	interest_id::INTEGER, 
	interest_name, 
	interest_summary, 
	composition, 
	index_value, 
	ranking, 
	percentile_ranking,
	month_year,
	created_at, 
	last_modified
FROM
	(
	SELECT
		*
	FROM
		interest_metrics
	WHERE
		interest_id NOT IN (
		SELECT
			interest_id
		FROM
			month_count)
) im
LEFT JOIN 
	interest_map i

ON 
	im.interest_id::INT = i.id
````

- Check new table

````sql
SELECT * FROM fresh_segments.segment_analysis
LIMIT 5
````

|interest_id|interest_name|interest_summary|composition|index_value|ranking|percentile_ranking|month_year|created_at|last_modified|
|-----------|-------------|----------------|-----------|-----------|-------|------------------|----------|----------|-------------|
|169|Social Liberals|People reading left-leaning social issues websites.|6.1|2.72|130|82.17|2018-07-01 00:00:00.000|2016-05-26 14:57:59.000|2018-05-23 11:30:13.000|
|101|Flower & Gift Basket Shoppers|Consumers shopping for flowers and gift baskets.|7.38|2.72|130|82.17|2018-07-01 00:00:00.000|2016-05-26 14:57:59.000|2018-05-23 11:30:12.000|
|6087|Metal and Rock Music Fans|People researching news and viewing online content focused on metal and rock music.|5.41|2.71|132|81.89|2018-07-01 00:00:00.000|2017-03-27 16:59:29.000|2017-12-27 12:31:08.000|
|104|Country Music Fans|People reading about country music and country music stars.|5.42|2.71|132|81.89|2018-07-01 00:00:00.000|2016-05-26 14:57:59.000|2018-05-23 11:30:12.000|
|6284|Gym Equipment Owners|People researching and comparing fitness trends and techniques. These consumers are more likely to spend money on gym equipment for their homes.|18.82|2.71|132|81.89|2018-07-01 00:00:00.000|2017-04-18 15:46:10.000|2017-12-27 12:31:08.000|

---

### 2. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year

- Calculate maximium composition for each interest

````sql 
WITH max_composition AS (
	SELECT 
		interest_id,
		MAX(composition) AS max_composition
	FROM
		fresh_segments.interest_metrics
	GROUP BY interest_id
)
````

- Select top 10 interests with lowest compostion value 

````sql 
SELECT 
	sa.interest_name,
	mc.max_composition,
	sa.month_year
FROM
	fresh_segments.segment_analysis sa
LEFT JOIN
	max_composition mc
ON
	sa.interest_id = mc.interest_id::INT
WHERE
	sa.composition = mc.max_composition
ORDER BY
	mc.max_composition ASC
LIMIT 10
````

|interest_name|max_composition|month_year|
|-------------|---------------|----------|
|Astrology Enthusiasts|1.88|2018-08-01 00:00:00.000|
|Medieval History Enthusiasts|1.94|2018-10-01 00:00:00.000|
|Dodge Vehicle Shoppers|1.97|2019-03-01 00:00:00.000|
|Xbox Enthusiasts|2.05|2018-07-01 00:00:00.000|
|Camaro Enthusiasts|2.08|2018-10-01 00:00:00.000|
|Budget Mobile Phone Researchers|2.09|2019-08-01 00:00:00.000|
|League of Legends Video Game Fans|2.09|2019-01-01 00:00:00.000|
|Super Mario Bros Fans|2.12|2018-07-01 00:00:00.000|
|Oakland Raiders Fans|2.14|2019-08-01 00:00:00.000|
|Budget Wireless Shoppers|2.18|2018-07-01 00:00:00.000|



- Select top 10 interests with lowest compostion value 

````sql
SELECT 
	sa.interest_name,
	mc.max_composition,
	sa.month_year
FROM
	fresh_segments.segment_analysis sa
LEFT JOIN
	max_composition mc
ON
	sa.interest_id = mc.interest_id::INT
WHERE
	sa.composition = mc.max_composition
ORDER BY
	mc.max_composition DESC
LIMIT 10
````

|interest_name|max_composition|month_year|
|-------------|---------------|----------|
|Work Comes First Travelers|21.2|2018-12-01 00:00:00.000|
|Gym Equipment Owners|18.82|2018-07-01 00:00:00.000|
|Furniture Shoppers|17.44|2018-07-01 00:00:00.000|
|Luxury Retail Shoppers|17.19|2018-07-01 00:00:00.000|
|Luxury Boutique Hotel Researchers|15.15|2018-10-01 00:00:00.000|
|Luxury Bedding Shoppers|15.05|2018-12-01 00:00:00.000|
|Shoe Shoppers|14.91|2018-07-01 00:00:00.000|
|Cosmetics and Beauty Shoppers|14.23|2018-07-01 00:00:00.000|
|Luxury Hotel Guests|14.1|2018-07-01 00:00:00.000|
|Luxury Retail Researchers|13.97|2018-07-01 00:00:00.000|

--- 

### 3. Which 5 interests had the lowest average `ranking` value?

````sql
SELECT 
	interest_name,
	ROUND(AVG(ranking), 0) AS avg_ranking
FROM
	fresh_segments.segment_analysis
GROUP BY
	interest_name
ORDER BY avg_ranking ASC 
LIMIT 5
````

|interest_name|avg_ranking|
|-------------|-----------|
|Winter Apparel Shoppers|1|
|Fitness Activity Tracker Users|4|
|Mens Shoe Shoppers|6|
|Shoe Shoppers|9|
|Competitive Tri-Athletes|12|

---

### 4. Which 5 interests had the largest standard deviation in their `percentile_ranking` value?


````sql
SELECT 
	interest_name,
	ROUND(STDDEV(percentile_ranking)::NUMERIC, 2) AS std_percentile_ranking
FROM
	fresh_segments.segment_analysis
GROUP BY
	interest_name
ORDER BY
	std_percentile_ranking DESC
LIMIT 5
````

|interest_name|std_percentile_ranking|
|-------------|----------------------|
|Techies|30.18|
|Entertainment Industry Decision Makers|28.97|
|Oregon Trip Planners|28.32|
|Personalized Gift Shoppers|26.24|
|Tampa and St Petersburg Trip Planners|25.61|

---




