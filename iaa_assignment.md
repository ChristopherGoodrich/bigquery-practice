## Context:

This is an assignment I completed for the Big Data class I took as part of the Masters of Science in Analytics at N.C. State's Institute for Advanced Analytics. The assignment was completed using the [Google BigQuery (free Sandbox version)](https://console.cloud.google.com/bigquery). The questions are answered first and then below is the SQL code and a snippet of the resulting query stored in Markdown tables.

---

</br>

## Data:

####  Austin bike share data

avaliable as public data in BigQuery (e.g.'bigquery-public-data.austin_bikeshare.bikeshare_stations')

</br>

* bikesharing_trips - each row represent ones ride including the start and end station, time, and duration of the ride
* bikesharing_stations - each row is one station and includes its name and geographic coordinates

---

</br>

## Questions:

</br>

**Question 1:** Which bike station in Austin is the most popular when starting a trip?

**Answer:** The most popular starting station is "21st & Speedyway @PCL".
 
</br>

################################################################################

</br>
 
**Question 2:** Which routes (start to end point) are the most popular. Please list the top 5. A route is a defined as a (start_station_name, end_station_name).

**Answer:** Table of top 5 routes

| **route** | **count** |
|---|---:|
| 21st & Speedway @PCL to 21st & Speedway @PCL | 13,517 |
| Riverside @ S. Lamar to Riverside @ S. Lamarr | 13,053 |
| 21st & Speedway @PCL to Dean Keeton & Speedway | 12,651 |
| Dean Keeton & Speedway to 21st & Speedway @PCL | 12,237 |
| Rainey St @ Cummings to Rainey St @ Cummings | 9,121 |

</br>

################################################################################

</br>

**Question 3:** Which stations are the furthest distance apart?

</br>

**Intepretation 1:** The furthest distance between stations that have a completed ride.

**Answer 3-1:** The furthest distance between stations for a completed trip is 8,246 meters (between Lake Austin & Enfeld and Capital Metro HQ).

<br/>

**Interpretation 2:** The furthest distance between any two stations even if there hasn't been a completed ride.

**Answer 3-2:** The furthest distance between any two stations is 8,600 meters (between Lake Austin & Enfeld and Lakeshore & Pleasant Valley)
    
---

</br>

## SQL Code:

**question 1:** most popular starting station

</br>

**GOOD query (see BAD query below for context):**

```
SELECT s.name AS start_station_name, COUNT(*) AS count
FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` as t
JOIN `bigquery-public-data.austin_bikeshare.bikeshare_stations` as s
  ON t.start_station_id=s.station_id
GROUP BY s.name
ORDER BY count DESC
LIMIT 3
```

**query results:**

| **start_station_name** | **count** |
|---|---|
| 21st & Speedway @PCL | 76,350 |
| Riverside @ S. Lamar | 42,442 |
| City Hall / Lavaca & 2nd | 37,111 |

</br>

**BAD query - lesson learned:** 

Below is a bad query. I originaly grouped on the **NAME** of a bike station in the trip data. It turns out that the station name field is not 1-to-1 with the station ID. Instead it is better to group by the station ID and then merge it to the station table to get the station name from there. The results are slightly different and it appears grouping by name does not give the correct count.

```
SELECT t.start_station_name, 
       count(t.start_station_name) as count
FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` as t
GROUP BY t.start_station_name
ORDER BY count DESC
LIMIT 3
```

**query results:**

| **start_station_name** | **count** |
|---|---|
| 21st & Speedway @PCL | 72,799 |
| Riverside @ S. Lamar | 40,635 |
| City Hall / Lavaca & 2nd | 36,520 |

</br>

################################################################################

</br>

**question 2:** top 5 routes

```
SELECT CONCAT(s1.name, ' to ', s2.name) AS route,
       COUNT(*) as count
FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS t
JOIN `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS s1
  ON t.start_station_id=s1.station_id
JOIN `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS s2
  ON SAFE_CAST(t.end_station_id AS int64)=s2.station_id
GROUP BY s1.name, s2.name
ORDER BY count DESC
LIMIT 5
```

**query results:**

| **route** | **count** |
|---|---:|
| 21st & Speedway @PCL to 21st & Speedway @PCL | 13,517 |
| Riverside @ S. Lamar to Riverside @ S. Lamarr | 13,053 |
| 21st & Speedway @PCL to Dean Keeton & Speedway | 12,651 |
| Dean Keeton & Speedway to 21st & Speedway @PCL | 12,237 |
| Rainey St @ Cummings to Rainey St @ Cummings | 9,121 |


</br>

################################################################################

</br>

**question 3 - intepretation 1:** furthest distance between stations with a completed ride between them

```
SELECT DISTINCT 
  st_s.name AS start_station_name,
  st_e.name AS end_station_name,
  ST_DISTANCE(ST_GEOGPOINT(st_s.longitude, st_s.latitude), 
              ST_GEOGPOINT(st_e.longitude, st_e.latitude)) AS dist
FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS t
JOIN `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS st_s
  ON t.start_station_id=st_s.station_id
JOIN `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS st_e
  ON SAFE_CAST(t.end_station_id AS INT64)=st_e.station_id
GROUP BY st_s.name, st_e.name, dist
ORDER BY dist DESC
LIMIT 3
```

**query results:**

| **start_station_name** | **end_station_name** | **distance** |
|---|---|---:|
| Capital Metro HQ - East 5th at Broadway | Lake Austin & Enfield | 8,246 |
| Lake Austin & Enfield | Capital Metro HQ - East 5th at Broadway | 8,246 |
| East 6th & Pedernales St. | Lake Austin & Enfield | 7,709 |

</br>

**question 3 - intepretation 2:** furthest distance between any two stations

```
SELECT t1.name AS s1_name,
       t2.name AS s2_name,
       ST_DISTANCE(ST_GEOGPOINT(t1.longitude, t1.latitude), 
                   ST_GEOGPOINT(t2.longitude, t2.latitude)) AS dist      
FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations` as t1
CROSS JOIN `bigquery-public-data.austin_bikeshare.bikeshare_stations` as t2
ORDER BY dist DESC
LIMIT 3
```

**query results:**

| **station 1** | **station 2** | **distance**|
|---|---|---:|
| Lake Austin & Enfield | Lakeshore & Pleasant Valley | 8,600 |
| Lakeshore & Pleasant Valley | Lake Austin & Enfield | 8,600 |
| Capital Metro HQ - East 5th at Broadway | Lake Austin & Enfield | 8,246 |
