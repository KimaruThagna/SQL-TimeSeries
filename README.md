# SQL-TimeSeries
SQL analysis of time series data using timeseriesDB for Postgres

# Data Source
For this tutorial, the nyc cab dataset was used. You can download from [here](https://timescaledata.blob.core.windows.net/datasets/nyc_data.tar.gz)

# Setup TimeSeriesDB for Postgres
This involves spinning up a postgres instance with the timeseries extension using docker.
 Run the below command to spin the container in detached mode

 ```
docker run -d --name timescaledb -p 5432:5432 -e POSTGRES_PASSWORD=password timescale/timescaledb:latest-pg12
 ```

 # Connect to DB instance using psql
 ```
docker exec -it timescaledb psql -U postgres
 ```

# Create database and the TimescaleDB extension
Run the SQL file `create_db_extention.sql`

# Copy data into DB
Since the data is on your local machine, you have to copy it into docker and then copy it into your table
To copy the dataset into docker, `docker cp nyc_data_rides.csv timescaledb:.`

To copy into the database, run the following command in your PSQL terminal `\COPY rides FROM nyc_data_rides.csv CSV;`

# Analysis
## Number of rides per day
`SELECT date_trunc('day', pickup_datetime) as day, COUNT(*) FROM rides GROUP BY day ORDER BY day;`

Using the time bucket function

 `SELECT time_bucket('1 day', pickup_datetime) as day, COUNT(*) FROM rides GROUP BY day ORDER BY day;`

## Average amount of fare for trips with 1 passenger within the first 10 days of 2016

```
SELECT date_trunc('day', pickup_datetime)
AS day, avg(fare_amount)
FROM rides
WHERE passenger_count = 1
AND pickup_datetime < '2016-01-10'
GROUP BY day ORDER BY day;
```

## Number of rides per rate type
```
SELECT rate_code, COUNT(vendor_id) AS num_trips
FROM rides
GROUP BY rate_code
ORDER BY rate_code;
```

## Number of rides per rate using descriptions and ranking

```
SELECT rates.description, COUNT(vendor_id) AS num_trips,
  RANK () OVER (ORDER BY COUNT(vendor_id) DESC) AS trip_rank FROM rides
  JOIN rates ON rides.rate_code = rates.rate_code
  GROUP BY rates.description
  ORDER BY trip_rank;
```

## Larger query
Find number of trips, avg duration, avg tip, avg cost, avg trip distance and avg number of passangers. All for rides to JWK and Newark only in January 2016

```
SELECT rates.description, COUNT(vendor_id) AS num_trips,
   AVG(dropoff_datetime - pickup_datetime) AS avg_trip_duration, AVG(total_amount) AS avg_total,
   AVG(tip_amount) AS avg_tip, MIN(trip_distance) AS min_distance, AVG (trip_distance) AS avg_distance, MAX(trip_distance) AS max_distance,
   AVG(passenger_count) AS avg_passengers
 FROM rides
 JOIN rates ON rides.rate_code = rates.rate_code
 WHERE rides.rate_code IN (2,3) AND pickup_datetime < '2016-02-01'
 GROUP BY rates.description
 ORDER BY rates.description;
```

Number of rides every 5 min on the first day

Using vanilla SQL
```
SELECT
  EXTRACT(hour from pickup_datetime) as hours,
  trunc(EXTRACT(minute from pickup_datetime) / 5)*5 AS five_mins,
  COUNT(*)
FROM rides
WHERE pickup_datetime < '2016-01-02 00:00'
GROUP BY hours, five_mins;

```

Using `time_buckets` 

```
SELECT time_bucket('5 minute', pickup_datetime) AS five_min, count(*)
FROM rides
WHERE pickup_datetime < '2016-01-02 00:00'
GROUP BY five_min
ORDER BY five_min;
```