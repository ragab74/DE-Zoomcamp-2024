# module1

## Question 1. Knowing docker tags

Run the command to get information on Docker

`docker --help`

Now run the command to get help on the "docker build" command:

`docker build --help`

Do the same for "docker run".

Which tag has the following text? - *Automatically remove the container when it exits*

- -delete
- -rc
- -rmc
- `-rm`

## Question 2. Understanding docker first run

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash. Now check the python modules that are installed ( use `pip list` ).

What is version of the package *wheel* ?

- `0.42.0`
- 1.0.0
- 23.0.1
- 58.1.0

## Question 3. Count records

How many taxi trips were totally made on September 18th 2019?

Tip: started and finished on 2019-09-18.

Remember that `**lpep_pickup_datetime**` and `**lpep_dropoff_datetime**` columns are in the format timestamp (date and hour + min + sec) and not in date.

- `15767`
- 15612
- 15859
- 89009

```sql
SELECT 
	CAST(lpep_pickup_datetime AS DATE) AS "day",
	COUNT(1)
FROM
	yellow_taxi_data 
WHERE
	CAST(lpep_pickup_datetime AS DATE) = '2019-09-18'
GROUP BY 1 ;
```

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

Tip: For every trip on a single day, we only care about the trip with the longest distance.

- 2019-09-18
- 2019-09-16
- `2019-09-26`
- 2019-09-21

```sql
SELECT 
	CAST(lpep_pickup_datetime AS DATE) AS "day",
	MAX(trip_distance) AS max_trip_distance
FROM
	yellow_taxi_data 

GROUP BY 1
ORDER BY 2 DESC LIMIT 1;
```

## Question 5. Three biggest pick up Boroughs

Consider `lpep_pickup_datetime`  in '2019-09-18' and ignoring Borough has Unknown

Which were the 3 pick up Boroughs that had a sum of `total_amount` superior to 50000?

- `"Brooklyn" "Manhattan" "Queens"`
- "Bronx" "Brooklyn" "Manhattan"
- "Bronx" "Manhattan" "Queens"
- "Brooklyn" "Queens" "Staten Island"

```sql
SELECT
    CAST(yt.lpep_pickup_datetime AS DATE) AS "day",
    COUNT(1) ,
    SUM(yt.total_amount) AS sum_total_amount,
    z."Borough"
FROM
    yellow_taxi_data yt JOIN zones z 
	  ON yt."PULocationID" = z."LocationID"
	
WHERE
    CAST(yt.lpep_pickup_datetime AS DATE) = '2019-09-18'
	
GROUP BY
    "day", z."Borough"
HAVING 
	SUM(yt.total_amount)> 50000 ;
```

## Question 6. Largest tip

For the passengers picked up in September 2019 in the zone name Astoria which was the drop off zone that had the largest tip? We want the name of the zone, not the id.

Note: it's not a typo, it's `tip` , not `trip`

- Central Park
- Jamaica
- `JFK Airport`
- Long Island City/Queens Plaza

```sql
WITH tips AS (
    SELECT
        yt.lpep_pickup_datetime AS pickup_datetime,
        yt."PULocationID" AS pickup_location_id,
        z1."Zone" AS pickup_zone,
        yt."DOLocationID" AS dropoff_location_id,
        z2."Zone" AS dropoff_zone,
        yt.tip_amount
    FROM
        yellow_taxi_data yt JOIN zones z1 ON yt."PULocationID" = z1."LocationID"
        JOIN zones z2 ON yt."DOLocationID" = z2."LocationID"

    WHERE
        CAST(yt.lpep_pickup_datetime AS DATE) BETWEEN '2019-09-01' AND '2019-09-30'
        AND z1."Zone" = 'Astoria'
)
SELECT
    pickup_zone, 
		dropoff_zone,
    MAX(tip_amount) AS max_tips	  
FROM
    tips
GROUP BY
    pickup_zone, dropoff_zone

ORDER BY max_tips DESC;
```
