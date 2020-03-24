## Context:

This is an assignment I completed for the Big Data class I took as part of the Masters of Science in Analytics at N.C. State's Institute for Advanced Analytics. The assignment was completed using the [Google BigQuery (free Sandbox version)](https://console.cloud.google.com/bigquery)

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

########################################

</br>
 
**Question 2:** Which routes (start to end point) are the most popular. Please list the top 5. A route is a defined as a (start_station_name, end_station_name).

**Answer:** Table of top 5 routes

| **route** | **count** |
|---|---:|
| 21st & Speedway @PCL to 21st & Speedway @PCL | 13,109 |
| Riverside @ S. Lamar to Riverside @ S. Lamarr | 12,302 |
| 21st & Speedway @PCL to Dean Keeton & Speedway | 10,173 |
| Dean Keeton & Speedway to 21st & Speedway @PCL | 9,523 |
| Rainey St @ Cummings to Rainey St @ Cummings | 8,676 |

</br>

########################################

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

**question 1:**

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

########################################

</br>

**question 2:**

```
SELECT route, COUNT(route) as count 
FROM (SELECT CONCAT(t.start_station_name, ' to ', t.end_station_name) AS route,
      FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS t)
GROUP BY route
ORDER BY count DESC
LIMIT 5
```

**query results:**

| **route** | **count** |
|---|---:|
| 21st & Speedway @PCL to 21st & Speedway @PCL | 13,109 |
| Riverside @ S. Lamar to Riverside @ S. Lamarr | 12,302 |
| 21st & Speedway @PCL to Dean Keeton & Speedway | 10,173 |
| Dean Keeton & Speedway to 21st & Speedway @PCL | 9,523 |
| Rainey St @ Cummings to Rainey St @ Cummings | 8,676 |


</br>

########################################

</br>

**question 3 - intepretation 1:** furthest distance between stations with a completed ride between them

```
SELECT DISTINCT 
  t.start_station_name AS s_st,
  t.end_station_name AS e_st,
  ST_DISTANCE(ST_GEOGPOINT(st_s.longitude, st_s.latitude), 
              ST_GEOGPOINT(st_e.longitude, st_e.latitude)) AS dist
FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS t
JOIN `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS st_s
  ON t.start_station_id=st_s.station_id
JOIN `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS st_e
  ON SAFE_CAST(t.end_station_id AS INT64)=st_e.station_id
GROUP BY s_st, e_st, dist
ORDER BY dist DESC
LIMIT 3
```

**query results:**

| **start station** | **end station** | **distance** |
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
