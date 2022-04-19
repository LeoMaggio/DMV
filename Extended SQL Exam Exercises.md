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
  SUM(revenue)*100/SUM(SUM(revenue)) OVER(PARTITION BY cName, city)
  RANK() OVER(ORDER BY SUM(numEmployees) DESC)
FROM CleaningServices CS, CleaningCompany C, Time T, Building B, Location L
WHERE CS.companyID = C.companyID
  AND CS.buildingID = B.buildingID
  AND CS.timeID = T.timeID
  AND CS.locationID = L.locationID
  AND year = 2019
GROUP BY cName, buildingID, city
```
