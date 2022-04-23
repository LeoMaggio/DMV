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
## February 01, 2021
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
## February 15, 2021
A plant species can be either an indoor or an outdoor plant. A plant species belongs to only one genus and one genus belong to only one family. A garden center can have 0 or more services. There are 4 available services: “parking”, “accessories shop”, “gardening shop” and “greenhouse”. Indoor, greenhouse, accessoryShop, gardenShop, and parking attributes can be “True” or “False”.
```
Gardens(TimeID, GardenCenterID, PlantId, numberOfPlants, revenue)
Time(TimeID, date, month, 2M, 3M, 4M, 6M, year, dayOfTheWeek, holiday)
GardenCenter(GardenCenterID, GardenCenter, city, province, region, greenhouse, accessoryShop, gardenShop, parking)
Plant(PlantID, plantSpecies, genus, family, indoor)
```
Separately for each plant species and month, compute the following metrics:
- the daily average number of plants
- the monthly percentage of the number of plants of the species with respect to the number of plants of the genus
- the cumulative total number of plants since the beginning of the year
```sql
SELECT plantSpecies, month,
  SUM(numberOfPlants)/COUNT(DISTINCT date),
  SUM(numberOfPlants)*100/SUM(SUM(numberOfPlants)) OVER(PARTITION BY month, genus),
  SUM(SUM(numberOfPlants)) OVER(PARTITION BY PlantID, plantSpecies, year
                                ORDER BY month
                                ROWS UNBOUNDED PRECEDING)
FROM Gardens G, Time T, Plant P
WHERE G.TimeID = T.TimeID
  AND G.PlantID = P.PlantID
GROUP BY PlantID, plantSpecies, month, year, genus
```
Consider only the garden centers having the “parking” service. Separately for each garden center and plant genus, compute the following metrics:
- the average revenue per plant
- the total revenues of the plant family, for each garden center
- assign a rank to each garden center within its province, based on its total revenues (rank 1st the garden center with the highest revenue in its province for each plant genus)
```sql
SELECT GardenCenter, genus,
  SUM(revenue)/SUM(numberOfPlants),
  SUM(SUM(revenue)) OVER(PARTITION BY GardenCenter, family),
  RANK() OVER(PARTITION BY province, genus
              ORDER BY SUM(revenue) DESC)
FROM Gardens G, Plant P, GardenCenter C
WHERE G.PlantID = P.PlantID
  AND G.GardenCenterID = C.GardenCenterID
  AND parking = TRUE
GROUP BY GardenCenter, genus, family, province
```
## June 17, 2021
- For each shipping, the departure and arrival cities, provinces and regions are recorded.
- For the shipping, the courier agency is recorded. The courier agency has a unique name. Each agency belongs to a Corporate group.
- The system stores the arrival date, the day of the week and if the day was an holiday or not. It also stores the month, year and semester.
```
CourierAgency(CourierAgencyID, CourierAgencyName, CorporateGroup)
Location(LocationID, city, province, region)
Time(TimeID, arrivalDate, dayOfTheWeek, holiday, month, 6M, year)
Shippings(CourierAgencyID, TimeID, ArrivalLocationID, DepartureLocationID, #packages, total_weight)
```
Separately for each courier agency and departure city, compute the following metrics:
- the percentage of packages with respect to the total number of packages of the agency for the departure region
- the average weight per package
- assign a rank to each courier agency within its corporate group, based on its total number of packages (rank 1st the courier agency with the highest number of shipped packages in its corporate group for each departure city)
```sql
SELECT CourierAgencyName, city,
  SUM(#packages)*100/SUM(SUM(#packages)) OVER(PARTITION BY CourierAgencyID, region),
  SUM(total_weight)/SUM(#packages),
  RANK() OVER(PARTITION BY CorporateGroup, city
              ORDER BY SUM(#packages) DESC)
FROM CourierAgency C, Location L, Shippings S
WHERE S.CourierAgencyID = C.CourierAgencyID
  AND S.DepartureLocationID = L.LocationID
GROUP BY CourierAgencyID, CourierAgencyName, DepartureLocationID, city, region, CorporateGroup
```
Separately for each month, departure province and arrival province, compute the following metrics:
- the daily average number of shipped packages
- the cumulative total weight of delivered packets since the beginning of the semester
```sql
SELECT month, L1.province, L2.province, 6M,
  SUM(#packages)/COUNT(DISTINCT date),
  SUM(SUM(total_weight)) OVER(PARTITION BY L1.province, L2.province, 6M
                              ORDER BY month
                              ROWS UNBOUNDED PRECEDING)
FROM Time T, Location L1, Location L2, Shippings S
WHERE S.TimeID = T.TimeID
  AND S.ArrivalLocationID = L1.LocationID
  AND S.DepartureLocationID = L2.LocationID
GROUP BY month, L1.province, L2.province, 6M
```
## September 01, 2021
- A video game has a specific name, a specific genre, and it is distributed by a video game company.
- A videogame can be appropriate for children or not.
  - The value of this field can be “0” for not appropriate and “1” for appropriate.
- A store is identified by a unique name. Stores are analyzed according to their city and country.
- The system records the sales with their date, the day of the week and if the day was an holiday or not. It also records the month, year, bimester, trimester and semester of the sales.
```
VideoGame(CodV, videoGameName, forChildren, genre, company)
Store(CodS, store, city, province, country)
Time(CodT, date, dayOfTheWeek, holiday, month, bimester, trimester, semester, year)
Fact(CodV, CodS, CodT, total_revenues, total_sold_videogames)
```
Separately for each videogame and store city, compute the following metrics:
- the percentage of copies of the videogame sold with respect to the total copies sold in the store province
- assign a rank to each videogame separately for video game company and city, based on its sales (rank 1st the video game with the highest number of sold copies for each city)
```sql
SELECT videoGameName, city,
  SUM(total_sold_videogames)*100/SUM(SUM(total_sold_videogames)) OVER(PARTITION BY CodV, province),
  RANK() OVER(PARTITION BY company, city
              ORDER BY SUM(total_sold_videogames) DESC)
FROM Fact F, VideoGame V, Store S
WHERE F.CodV = V.CodV
  AND F.CodS = S.CodS
GROUP BY CodV, videoGameName, city, province, company
```
Separately for each store city and bimester, compute the following metrics, only for the videogames appropriate for children:
- the cumulative revenues since the beginning of the semester
- the daily average revenues
- the percentage of revenues in the bimester with respect to the revenues in the semester, for each city
```sql
SELECT city, bimester,
  SUM(SUM(total_revenues)) OVER(PARTITION BY semester, city
                                ORDER BY bimester
                                ROWS UNBOUNDED PRECEDING)
  SUM(total_revenues)/COUNT(DISTINCT date)
  SUM(total_revenues)*100/SUM(SUM(total_revenues)) OVER(PARTITION BY semester, city)
FROM Fact F, VideoGame V, Store S, Time T
WHERE F.CodV = V.CodV
  AND F.CodS = S.CodS
  AND F.CodT = T.CodT
  AND forChildren = 1
GROUP BY city, bimester, semester
```
## January 28, 2022
The name of the renting point is unique. The ChildrenBike is True if the bike is for children, False otherwise.
```
Bike(CodM, BikeModel, Producer, ChildrenBike)
RentingPoint(CodP, Name, City, Region)
Time(CodT, Date, dayOfTheWeek, holiday, month, 2M, 3M, 4M, year)
Fact(CodM, CodP, CodT, total_rentings, total_rented_bikes, total_income)
```
Separately for each Renting Point and trimester, compute:
- the average number of rented bikes per renting
- the cumulative total number of rentings since the beginning of the year
- for each region, assign a rank to the renting points based on the total rentings (rank 1st the highest number), separately for each trimester
```sql
SELECT Name, trimester,
  SUM(total_rented_bikes)/SUM(total_rentings),
  SUM(SUM(total_rentings)) OVER(PARTITION BY Name, year
                                ORDER BY trimester
                                ROWS UNBOUNDED PRECEDING)
  RANK() OVER(PARTITION BY region, trimester
              ORDER BY SUM(total_rentings) DESC)
FROM Fact F, RentingPoint R, Time T
WHERE F.CodP = R.CodP
  AND F.CodT = T.CodT
GROUP BY CodP, Name, trimester, year, region
```
Separately for each city, producer, and month, compute:
- the percentage of incomes in each month and city of each producer, with respect to the producer yearly total for the city
- the average income per rented bike
- for each city, assign a rank to the producer based on the income (rank 1st the highest income), separately for each month
```sql
SELECT city, producer, month,
  SUM(total_income)*100/SUM(SUM(total_income)) OVER(PARTITION BY producer, city, year),
  SUM(total_income)/SUM(total_rented_bikes),
  RANK() OVER(PARTITION BY city, month
              ORDER BY SUM(total_income) DESC)
FROM Fact F, RentingPoint R, Time T, Bike B
WHERE F.CodP = R.CodP
  AND F.CodT = T.CodT
  AND F.CodM = B.CodM
GROUP BY city, producer, month, year
```
