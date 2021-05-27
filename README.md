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

