# Project: Fresh Segments

<img src="https://user-images.githubusercontent.com/35038779/219868785-3ce39f0f-60fc-4a59-8971-0570db4497ae.png" alt="Image" width="500" height="520">

# Overview

Fresh Segments is a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

The goal of the project is to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

The project is based on [SQL-8-week-challenge](https://8weeksqlchallenge.com/case-study-8/) case. 


## Contents

- [Project workflow](#project-workflow)
- [Data description](#data-description)
- [Modules and tools](#modules-and-tools)


### Project workflow

  1. Setting up PostgresSQL server 
  2. [Create project database and load schemas with input tables](https://github.com/LtvnSergey/SQL-Fresh-sSegments/blob/main/input.md)
  3. [Data exploration and cleansing](https://github.com/LtvnSergey/SQL-Fresh-Segments/blob/main/analysis/1_data_exploration_and_cleansing.md)
  4. [Interest analysis](https://github.com/LtvnSergey/SQL-Fresh-Segments/blob/main/analysis/2_interest_analysis.md)
  5. [Segment analysis](https://github.com/LtvnSergey/SQL-Fresh-Segments/blob/main/analysis/3_segment_analysis.md)
  6. [Index analysis](https://github.com/LtvnSergey/SQL-Fresh-Segments/blob/main/analysis/4_index_analysis)


### Data description

---

* **Interest Metrics**


This table contains information about aggregated interest metrics for a specific major client of Fresh Segments which makes up a large proportion of their customer base.

Each record in this table represents the performance of a specific `interest_id` based on the client’s customer base interest measured through clicks and interactions with specific targeted advertising content.


| _month | _year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
| ------ | ----- | ---------- | ----------- | ----------- | ----------- | ------- | ------------------ |
| 7      | 2018  | 07-2018    | 32486       | 11.89       | 6.19        | 1       | 99.86              |
| 7      | 2018  | 07-2018    | 6106        | 9.93        | 5.31        | 2       | 99.73              |
| 7      | 2018  | 07-2018    | 18923       | 10.85       | 5.29        | 3       | 99.59              |
| 7      | 2018  | 07-2018    | 6344        | 10.32       | 5.1         | 4       | 99.45              |
| 7      | 2018  | 07-2018    | 100         | 10.77       | 5.04        | 5       | 99.31              |

---

* **Interest Map**


This mapping table links the `interest_id` with their relevant interest information. You will need to join this table onto the previous `interest_details` table to obtain the `interest_name` as well as any details about the summary information.

| id  | interest_name             | interest_summary                                             | created_at               | last_modified            |
| --- | ------------------------- | ------------------------------------------------------------ | ------------------------ | ------------------------ |
| 1   | Fitness Enthusiasts       | Consumers using fitness tracking apps and websites.          | 2016-05-26T14:57:59.000Z | 2018-05-23T11:30:12.000Z |
| 2   | Gamers                    | Consumers researching game reviews and cheat codes.          | 2016-05-26T14:57:59.000Z | 2018-05-23T11:30:12.000Z |
| 3   | Car Enthusiasts           | Readers of automotive news and car reviews.                  | 2016-05-26T14:57:59.000Z | 2018-05-23T11:30:12.000Z |
| 4   | Luxury Retail Researchers | Consumers researching luxury product reviews and gift ideas. | 2016-05-26T14:57:59.000Z | 2018-05-23T11:30:12.000Z |
| 5   | Brides & Wedding Planners | People researching wedding ideas and vendors.                | 2016-05-26T14:57:59.000Z | 2018-05-23T11:30:12.000Z |

---

* **Entity Relationship Diagram**:

<img src="https://user-images.githubusercontent.com/35038779/219869820-6466b536-aad6-4ad1-a700-378ebfd5c888.png" alt="Image" width="800" height="520">

---

### Modules and tools

PostgresSQL | PgAdmin 4
