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
	mc.max_composition AS min_composition
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

|interest_name|min_composition|month_year|
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
ORDER BY avg_ranking DESC 
LIMIT 5
````

|interest_name|avg_ranking|
|-------------|-----------|
|League of Legends Video Game Fans|1037|
|Computer Processor and Data Center Decision Makers|974|
|Astrology Enthusiasts|969|
|Medieval History Enthusiasts|962|
|Budget Mobile Phone Researchers|961|

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


### 5. For the 5 interests found in the previous question - what was minimum and maximum `percentile_ranking` values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?


````sql
WITH highest_std_perc_rank AS (
SELECT 
		interest_id,
		ROUND(STDDEV(percentile_ranking)::NUMERIC, 2) AS std_percentile_ranking
FROM
		fresh_segments.segment_analysis
GROUP BY
		interest_id
ORDER BY
		STDDEV(percentile_ranking) DESC
LIMIT 5
),

min_max_perc_ranking AS (
SELECT 
		interest_id, 
		MIN(percentile_ranking) AS min_perc_ranking,
		MAX(percentile_ranking) AS max_perc_ranking
FROM
		fresh_segments.segment_analysis
GROUP BY
		interest_id
HAVING
		interest_id IN (
	SELECT
			interest_id
	FROM
			highest_std_perc_rank)
)

SELECT 
	min_rank.interest_name,
	min_rank.min_perc_ranking,
	min_rank.min_perc_ranking_date,
	max_rank.max_perc_ranking,
	max_rank.max_perc_ranking_date
FROM
	(
	SELECT
		sa.interest_name,
		min_perc_ranking,
		month_year::DATE AS min_perc_ranking_date
	FROM
		fresh_segments.segment_analysis sa
	INNER JOIN
	min_max_perc_ranking r
ON
		sa.interest_id = r.interest_id
	WHERE
		sa.percentile_ranking = r.min_perc_ranking
) min_rank
JOIN 
(
	SELECT
		sa.interest_name,
		max_perc_ranking,
		month_year::DATE AS max_perc_ranking_date
	FROM
		fresh_segments.segment_analysis sa
	INNER JOIN
	min_max_perc_ranking r
ON
		sa.interest_id = r.interest_id
	WHERE
		sa.percentile_ranking = r.max_perc_ranking
) max_rank
ON
	min_rank.interest_name = max_rank.interest_name
````

|interest_name|min_perc_ranking|min_perc_ranking_date|max_perc_ranking|max_perc_ranking_date|
|-------------|----------------|---------------------|----------------|---------------------|
|Tampa and St Petersburg Trip Planners|4.84|2019-03-01|75.03|2018-07-01|
|Oregon Trip Planners|2.2|2019-07-01|82.44|2018-11-01|
|Personalized Gift Shoppers|5.7|2019-06-01|73.15|2019-03-01|
|Techies|7.92|2019-08-01|86.69|2018-07-01|
|Entertainment Industry Decision Makers|11.23|2019-08-01|86.15|2018-07-01|

-  From the table above we can see that for whose interests `ranking percentile` become signinificantly lower from 86 - 73 % to 11 - 2 % in a matter of several months which means `index_value` became lower as well.


### 6. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?

- Based on the `composition` values and `ranking` we can say that 
	1. Avoid showing topics related to games/astrology/history/tech stuff
	2. Most popular topics are related to luxury things/gym-fitness/travel
	
- Topics with largest standart deviation of `percentile_ranking` could have vary differently in terms of their intereting to customers 
from month to month. Such interests like 'Oregon Trip Planners' and 'Tampa and St Petersburg Trip Planners' sould be quite situational to use.

- From maximum and minimum `percentile_ranking' table we can when such topics could be most interesting

- Perhaps we could assume that most of our customers are middle-high class who are interested in luxury stuff, body appearence (gym, fitness) and occasional travelling 
