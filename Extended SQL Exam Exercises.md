# Extended SQL Exam Exercises
## January 31, 2020
```
CleaningServices (companyID, timeID, buildingID, serviceID, 
  numHours, revenue, numEmployees, personnel_cost)
Location (locationID, city, province, region)
Time (timeID, date, dayOfWeek, month, monthOfYear, 2m, 3m, 4m, 6m, year)
CleaningCompany (companyID, cName, C_locationID, steam_cleaners, floor_scrubbers, wet_vacuums, floor_polishers, ..., escalator_cleaners)
Building (buildingID, bName, bType, B_LId)
Services(serviceID, sName)
```
Separately for each cleaning company, type of service, and month, analyze:
- the average daily revenue
- the cumulative revenue since the beginning of the year
- the percentage of hours of each service type with respect to the the total number of hours across all service types
```sql
SELECT cName, sName, year, month,
  SUM(revenue)/COUNT(DISTINCT date),
  SUM(SUM(revenue)) OVER(PARTITION BY cName, sName, year
                         ORDER BY month
                         ROWS UNBOUNDED PRECEDING)
  SUM(numHours)*100/SUM(SUM(numHours)) OVER(PARTITION BY month, cName)
FROM CleaningServices CS, CleaningCompany C, Services S, Time T
WHERE CS.companyID = C.companyID
  AND CS.serviceID = S.serviceID
  AND CS.timeID = T.timeID
GROUP BY cName, sName, year, month
```
Considering the cleaning services of 2019, separately for each building and cleaning company, analyze:
- the profits (i.e., the difference between revenues and costs)
- the percentage of revenues with respect to the total revenues of the company for all the buildings in the same city
- assign a rank according to the total number of employees (rank 1st the highest)
```sql
SELECT cName, buildingID,
  SUM(revenue) - SUM(personnel_cost),
  SUM(revenue)*100/SUM(SUM(revenue)) OVER(PARTITION BY cName, city),
  RANK() OVER(ORDER BY SUM(numEmployees) DESC)
FROM CleaningServices CS, CleaningCompany C, Time T, Building B, Location L
WHERE CS.companyID = C.companyID
  AND CS.buildingID = B.buildingID
  AND CS.timeID = T.timeID
  AND CS.locationID = L.locationID
  AND year = 2019
GROUP BY cName, buildingID, city
```
## February 14, 2020
```
GymManager (LocationID, TimeID, HourID, GymID, ConfID, SubscriptionID, 
  numEntrances, numSubscriptions, totRevenues)
Location (LocationID, city, province, region)
Time (TimeID, date, dayOfWeek, holiday, month, MoY, 2m, 3m, 4m, 6m, year)
Hour (HourID, hourOfTheDay)
Gym (GymID, gymName, LocationID, indoor_swimming_pool, outdoor_swimming_pool, spa, ..., sauna)
UserConfig (ConfID, ageGroup, sex)
Subscription (SubscriptionID, subscriptionType)
```
Considering only the gyms with outdoor or indoor swimming pools, separately for each year, subscription type, and city of the gym, analyze:
- the percentage of subscriptions with respect to the total number of subscriptions for all the gyms of the same region
- the cumulative yearly revenues
- assign a rank according to the total number of subscriptions (rank 1st the highest)
```sql
SELECT year, subscriptionType, city, region,
  SUM(numSubscriptions)*100/SUM(SUM(numSubscriptions)) OVER(PARTITION BY year, region),
  SUM(SUM(totRevenues)) OVER(PARTITION BY subscriptionType, city
                             ORDER BY year
                             ROWS UNBOUNDED PRECEDING)
  RANK() OVER(ORDER BY SUM(numSubscriptions) DESC)
FROM Gym G, Location L, Time T, Subscription S, GymManager GM
WHERE GM.TimeID = T.TimeID
  AND GM.GymID = G.GymID
  AND GM.SubscriptionID = S.SubscriptionID
  AND G.LocationID = L.LocationID
  AND (indoor_swimming_pool = TRUE OR outdoor_swimming_pool = TRUE)
GROUP BY year, subscriptionType, city, region
```
Separately for each age group and city of birth of the users, analyze:
- the hourly average number of entrances
- the percentage of revenues with respect to the total revenues across all cities of birth
- the average revenue per subscription
- the average revenue per entrance
```sql
SELECT ageGroup, city,
  SUM(numEntrances)/COUNT(DISTINCT date, hourOfTheDay),
  SUM(totRevenues)*100/SUM(SUM(totRevenues)) OVER(PARTITION BY ageGroup),
  SUM(totRevenues)/SUM(numSubscriptions),
  SUM(totRevenues)/SUM(numEntrances)
FROM Location L, GymManager GM, UserConfig U, Hour H, Time T
WHERE GM.TimeID = T.TimeID
  AND GM.HourID = H.HourID
  AND GM.ConfID = U.ConfID
  AND GM.LocationID = L.LocationID
GROUP BY ageGroup, city
```
## June 18, 2020
```
Purchase(TimeID, StoreID, income, number_of_items_sold)
Store(StoreID, name, brand, city, region)
Date (TimeID, date, month, year)
```
Separately for each city and month, compute:
- the total income
- the percentage of income in each month, with respect to the total of the year
- inside each region, assign a rank to the cities based on the income (rank 1st the highest income city), separately for each month
```sql
SELECT city, month,
  SUM(income),
  SUM(income)*100/SUM(SUM(income)) OVER(PARTITION BY city, year),
  RANK() OVER(PARTITION BY month, region
              ORDER BY SUM(income) DESC)
FROM Purchase P, Store S, Date D
WHERE P.TimeID = D.TimeID
  AND P.StoreID = S.StoreID
GROUP BY city, month, year, region
```
Separately for each month and brand, compute:
- the average income per item
- the daily average number of items
- the cumulative total number of items since the beginning of the year
```sql
SELECT brand, month, year
  SUM(income)/SUM(number_of_items_sold),
  SUM(number_of_items_sold)/COUNT(DISTINCT date),
  SUM(SUM(number_of_items_sold)) OVER(PARTITION BY brand, year
                                      ORDER BY month
                                      ROWS UNBOUNDED PRECEDING)
FROM Purchase P, Store S, Date D
WHERE P.TimeID = D.TimeID
  AND P.StoreID = S.StoreID
GROUP BY brand, month, year
```
## Februrary 01, 2021
```
MusicStreaming(TimeID, SongID, PlatformID, NumberOfStreamings, NumberOfLikes)
Time(TimeID, date, month, 2M, 3M, 6M, year, dayOfTheWeek)
Song(SongID, Song, album, classic, indie, pop, .., rock)
UserLocation(UserLocationID, province, region, country)
```
For each song and month, compute the following metrics:
- the total number of streamings
- the cumulative total number of streamings since the beginning of the year
- assign a rank to each song, separately for each album, based on the monthly number of streamings (rank 1st the most streamed song of the album for each month)
```sql
SELECT song, month,
  SUM(NumberOfStreamings),
  SUM(SUM(NumberOfStreamings)) OVER(PARTITION BY songID, year
                                    ORDER BY month
                                    ROWS UNBOUNDED PRECEDING)
  RANK() OVER(PARTITION BY album, month
              ORDER BY SUM(NumberOfStreamings) DESC)
FROM MusicStreaming M, Song S, Time T
WHERE M.SongID = S.SongID
  AND M.TimeID = T.TimeID
GROUP BY song, songID, month, year, album
```
Separately for each song and province of the user, compute the following metrics:
- the average number of monthly likes
- the percentage of the number of likes with respect to the total number of likes received by users in the same country
- the number of likes of the album in the user province
```sql
SELECT song, province,
  SUM(NumberOfLikes)/COUNT(DISTINCT month)
  SUM(NumberOfLikes)*100/SUM(SUM(NumberOfLikes)) OVER(PARTITION BY songID, country)
  SUM(SUM(NumberOfLikes)) OVER(PARTITION BY album, province)
FROM MusicStreaming M, Song S, Time T, UserLocation U
WHERE M.SongID = S.SongID
  AND M.TimeID = T.TimeID
  AND M.UserLocationId = U.UserLocationId
GROUP BY song, songID, province, country, album
```
