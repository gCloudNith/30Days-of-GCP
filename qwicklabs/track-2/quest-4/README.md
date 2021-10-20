<h1 align="center">Task 1: Clean your training data</h1>

<h3>Using the cloud console</h3>
Navigation menu -> BigQuery

In the Query editor run the following query to create a table and clean the data

```
CREATE OR REPLACE TABLE
  taxirides.taxi_training_data AS
SELECT
  (tolls_amount + fare_amount) AS fare_amount,
  pickup_datetime,
  pickup_longitude AS pickuplon,
  pickup_latitude AS pickuplat,
  dropoff_longitude AS dropofflon,
  dropoff_latitude AS dropofflat,
  passenger_count AS passengers,
FROM
  taxirides.historical_taxi_rides_raw
WHERE
  RAND() < 0.001
  AND trip_distance > 0
  AND fare_amount >= 2.5
  AND pickup_longitude > -78
  AND pickup_longitude < -70
  AND dropoff_longitude > -78
  AND dropoff_longitude < -70
  AND pickup_latitude > 37
  AND pickup_latitude < 45
  AND dropoff_latitude > 37
  AND dropoff_latitude < 45
  AND passenger_count > 0
```

<h1 align="center">Task 2: Create a BQML model called taxirides.fare_model</h1>

```
CREATE or REPLACE MODEL
  taxirides.fare_model OPTIONS (model_type='linear_reg',
    labels=['fare_amount']) AS
WITH
  taxitrips AS (
  SELECT
    *,
    ST_Distance(ST_GeogPoint(pickuplon, pickuplat), ST_GeogPoint(dropofflon, dropofflat)) AS euclidean
  FROM
    `taxirides.taxi_training_data` )
  SELECT
    *
  FROM
    taxitrips
```    

Your model will need an RMSE of 10 or less to complete the task. To evaluate the Root Mean Square Error (RMSE) of the prediction model using the following query


```
#standardSQL
SELECT
  SQRT(mean_squared_error) AS rmse
FROM
  ML.EVALUATE(MODEL taxirides.fare_model,
    (
    WITH
      taxitrips AS (
      SELECT
        *,
        ST_Distance(ST_GeogPoint(pickuplon, pickuplat), ST_GeogPoint(dropofflon, dropofflat)) AS euclidean
      FROM
        `taxirides.taxi_training_data` )
      SELECT
        *
      FROM
        taxitrips ))
```

<h1 align="center">Task 3: Perform a batch prediction on new data</h1>

In the query editor, run the following query to use BQML model to predict taxi fares

```
CREATE OR REPLACE TABLE taxirides.2015_fare_amount_predictions
  AS
SELECT * FROM ML.PREDICT(MODEL taxirides.fare_model,(
  SELECT * FROM taxirides.report_prediction_data)
)
```

