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

* **Users**

Customers who visit the Clique Bait website are tagged via their `cookie_id`.

![image](https://user-images.githubusercontent.com/35038779/217028827-09b348c0-737f-47fc-9ae8-f27eb7c8f73c.png)

* **Events**

Customer visits are logged in this events table at a `cookie_id` level and the `event_type` and `page_id` values can be used to join onto relevant satellite tables to obtain further information about each event.

The sequence_number is used to order the events within each visit.

![image](https://user-images.githubusercontent.com/35038779/217029123-2695dc66-1b27-42b1-a7e5-b199a0e55aaf.png)


* **Event Identifier** 

The `event_identifier` table shows the types of events which are captured by Clique Bait’s digital data systems.

![image](https://user-images.githubusercontent.com/35038779/217029809-bbfca0d9-0f60-4f2a-8235-6fd3e816da02.png)


* **Campaign Identifier**

This table shows information for the 3 campaigns that Clique Bait has ran on their website so far in 2020.

![image](https://user-images.githubusercontent.com/35038779/217030051-2493d98d-4ccf-4fd6-9adf-6e9804313f2f.png)


* **Page Hierarchy**

This table lists all of the pages on the Clique Bait website which are tagged and have data passing through from user interaction events.

![image](https://user-images.githubusercontent.com/35038779/217030242-6d7a50ac-dabb-4948-a207-131ec1424406.png)



* **Entity Relationship Diagram**:

<img src="https://user-images.githubusercontent.com/35038779/217026508-fbcf5de1-463b-4450-8dd4-9c07aeaac714.png" alt="Image" width="1500" height="520">


### Modules and tools

PostgresSQL | PgAdmin 4
