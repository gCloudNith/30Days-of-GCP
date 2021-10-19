#  Insights from Data with BigQuery

- Open Public Dataset
- In the GCP Console go to ```Navigation Menu``` > ```BigQuery```.
- Click on ```+ ADD DATA```.
- Then ```Explore public datasets``` from the left pane.
- Search ```covid19_open_data``` and select it.

## Step 1: Total Confirmed Cases
- Copy and paste the following code to the Query editor and then click Run.
```
SELECT
  SUM(cumulative_confirmed) AS total_cases_worldwide
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  date = "2020-04-15"
```
The query should return a single row containing the sum of confirmed cases across all countries.

## Step 2: Worst Affected Areas
- Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
```
with deaths_by_states as (
SELECT subregion1_name as state, sum(cumulative_deceased) as death_count 
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` where country_name="United States of America" and date='2020-04-10' and subregion1_name is NOT NULL 
group by subregion1_name ) 
select count(*) as count_of_states from deaths_by_states where death_count > 100
```
The query should list the output in the field count_of_states.

## Step 3: Identifying Hotspots
```
Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
SELECT * FROM ( 
SELECT subregion1_name as state, sum(cumulative_confirmed) as total_confirmed_cases 
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` WHERE country_code="US" AND date='2020-04-10' AND subregion1_name is NOT NULL 
GROUP BY subregion1_name ORDER BY total_confirmed_cases DESC ) WHERE total_confirmed_cases > 1000
```
The query should return the State Name and the corresponding confirmed cases arranged in descending order.

## Step 4: Fatality Ratio
- Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
```
SELECT SUM(cumulative_confirmed) AS total_confirmed_cases, SUM(cumulative_deceased) AS total_deaths, (SUM(cumulative_deceased)/SUM(cumulative_confirmed))*100 AS case_fatality_ratio
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name="Italy" AND date BETWEEN "2020-04-01" AND "2020-04-30"
```
## Step 5: Identifying specific day
- Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
```
SELECT
 date
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
 country_name = 'Italy'
 AND cumulative_deceased > 10000
ORDER BY date
LIMIT 1
```
The query should return the date in the format yyyy-mm-dd.

## Step 6: Finding days with zero net new cases
- Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
```
WITH india_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="India"
    AND date between '2020-02-21' and '2020-03-15'
  GROUP BY
    date
  ORDER BY
    date ASC
 )

, india_previous_day_comparison AS
(SELECT
  date,
  cases,
  LAG(cases) OVER(ORDER BY date) AS previous_day,
  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases
FROM india_cases_by_date
)
SELECT
  COUNT(date)
FROM
  india_previous_day_comparison
WHERE
  net_new_cases = 0
```
The query should return the number of days in India between 21 Feb 2020 and 15 March 2020 when there were zero increases in the number of confirmed cases.

## Step 7: Doubling rate
Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
```
WITH us_cases_by_date AS (
  SELECT
    date,
    SUM( cumulative_confirmed ) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="United States of America"
    AND date between '2020-03-22' and '2020-04-20'
  GROUP BY
    date
  ORDER BY
    date ASC
 )

, us_previous_day_comparison AS
(SELECT
  date,
  cases,
  LAG(cases) OVER(ORDER BY date) AS previous_day,
  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases,
  (cases - LAG(cases) OVER(ORDER BY date))*100/LAG(cases) OVER(ORDER BY date) AS percentage_increase
FROM us_cases_by_date
)
SELECT
  Date,
  cases AS Confirmed_Cases_On_Day,
  previous_day AS Confirmed_Cases_Previous_Day,
  percentage_increase AS Percentage_Increase_In_Cases
FROM
  us_previous_day_comparison
WHERE
  percentage_increase > 10
The query should return the list of dates, the confirmed cases on that day, the confirmed cases the previous day, and the percentage increase in cases between the days.
```
## Step 8: Recovery rate
- Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
```
WITH cases_by_country AS (
  SELECT
    country_name AS country,
    SUM(cumulative_confirmed) AS cases,
    SUM(cumulative_recovered) AS recovered_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    date="2020-05-10"
  GROUP BY
    country_name
)

, recovered_rate AS (
  SELECT
    country, cases, recovered_cases,
    (recovered_cases * 100)/cases AS recovery_rate
  FROM
    cases_by_country
)

SELECT country, cases AS confirmed_cases, recovered_cases, recovery_rate
FROM
   recovered_rate
WHERE
   cases > 50000
ORDER BY recovery_rate DESC
LIMIT 10
```
The query should return the recovery rates of countries arranged in descending order (limit to 10) on the date May 10, 2020.

## Step 9: CDGR — Cumulative Daily Growth Rate
- Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
```
WITH
  france_cases AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS total_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="France"
    AND date IN ('2020-01-24',
      '2020-05-10')
  GROUP BY
    date
  ORDER BY
    date)
, summary as (
SELECT
  total_cases AS first_day_cases,
  LEAD(total_cases) OVER(ORDER BY date) AS last_day_cases,
  DATE_DIFF(LEAD(date) OVER(ORDER BY date),date, day) AS days_diff
FROM
  france_cases
LIMIT 1
)

select first_day_cases, last_day_cases, days_diff, POWER(last_day_cases/first_day_cases,1/days_diff)-1 as cdgr
from summary
```
The query should calculate the CDGR on May 10, 2020(Cumulative Daily Growth Rate) for France since the day the first case was reported.

## Step 9: Create a Datastudio report
- Click on compose new query and then copy and paste the following query into the BigQuery Query editor.
```
SELECT
  date, SUM(cumulative_confirmed) AS country_cases,
  SUM(cumulative_deceased) AS country_deaths
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  date BETWEEN '2020-03-15'
  AND '2020-04-30'
  AND country_name='United States of America'
GROUP BY date
```
- Open Data Studio using Gcloud credentials.

Note: If you are accessing Data studio for the first time then you might be asked to fill your basic info and mailing preferences. Please fill your country name, accept the terms and conditions and select No for all 3 mailing preferences.

- Now open a blank report.

- Click on add Data to report > BigQuery

- Authorize

- Select custom Query > YOUR_GCP_PROJECT_ID and enter the following query
```
SELECT
  date, SUM(cumulative_confirmed) AS country_cases,
  SUM(cumulative_deceased) AS country_deaths
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  date BETWEEN '2020-03-15'
  AND '2020-04-30'
  AND country_name='United States of America'
GROUP BY date
```
The data must be added to your report.

- In the Data Studio report navigation bar, select ```Add a chart``` > ```Time-series Chart```.

- Add ```country_cases``` and ```country_deaths``` to the ```Metric``` field.

- Your graph must resemble the one shown below:

![](./Resources/images/t2q3.png)

- Click on ```Save``` to commit the change.

Congratulations! You have completed your lab.
