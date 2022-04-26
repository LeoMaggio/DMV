# Extended SQL Exercises
## 1. Customers
The following relations are given:
```
Customer(CustomerID, CustomerSurname, Province, Region)
Category(CategoryID, CategoryName)
Agent(AgentID, AgentSurname, Agency)
Time(TimeID, Month, Quarter, Semester, Year)
Sales(TimeID, CustomerID, ItemCategoryID, AgentID, TotAmount, NumSoldItems, TotDiscount)
```
### 1.1 
Show for each category of items,
- The category
- The total sale amount for the category
- The total number of sold items within the category
- The rank of the category according to its total sale amount
- The rank of the category according to its total number of sold items
- Sort the result for increasing value of total number of sold items within the category
```sql
SELECT CategoryName,
  SUM(TotAmount),
  SUM(NumSoldItems),
  RANK() OVER(ORDER BY SUM(TotAmount) DESC) AS RANK_TotAmount,
  RANK() OVER(ORDER BY SUM(NumSoldItems) DESC) AS RANK_NumSoldItems
FROM Category C, Sales S
WHERE C.CategoryID = S.CategoryID
GROUP BY CategoryName, CategoryID
ORDER BY RANK_NumSoldItems
```
### 1.2
Show, for each province
- The province
- The region of the province
- The total sale amount for the province
- The rank of the province according to its total sale amount, separately for each region
```sql
SELECT Province, Region,
  SUM(TotAmount),
  RANK() OVER(PARTITION BY Region
              ORDER BY SUM(TotAmount) DESC) AS RANK_TotAmount
FROM Customer C, Sales S
WHERE C.CustomerID = S.CustomerID
GROUP BY Province, Region
```
### 1.3
Show, for each province and month
- The province
- The region of the province
- The month
- The total sale amount for the province in the current month
- The rank of the province according to its total sale amount, separately for each moth
```sql
SELECT Province, Region, Month,
  SUM(TotAmount),
  RANK() OVER(PARTITION BY Month
              ORDER BY SUM(TotAmount) DESC) AS RANK_TotAmount
FROM Customer C, Sales S, Time T
WHERE C.CustomerID = S.CustomerID
  AND T.TimeID = S.TimeID
GROUP BY Province, Region, Month
```
### 1.4
Show, for each region and month
- The region
- The month
- The total sale amount for the region in the current month
- The cumulative sale amount for increasing months, separately for each region
```sql
SELECT Region, Month,
  SUM(TotAmount),
  SUM(SUM(TotAmount)) OVER(PARTITION BY Region
                           ORDER BY Month
                           ROWS UNBOUNDED PRECEDING) AS CUM_TotAmount
FROM Customer C, Sales S, Time T
WHERE C.CustomerID = S.CustomerID
  AND T.TimeID = S.TimeID
GROUP BY Region, Month
```
## 2. Rentals
The following relations are given on rentals of different types of objects for a seaside resort:
```
OBJECTS (ObjectID, ObjectType)
CUSTOMERS (CustomerID, Name, Surname)
TIME (TimeID, Date, Month, Year)
RENTALS (TimeID, CustomerID, ObjectID, Price)
```
### 2.1
Show, for each object
- The object code
- The object type
- The object total rentals (number of times the object has been rented)
- The object total income (SUM(price))
- The object rank according to its total rentals
- The object rank according to its total income
```sql
SELECT ObjectID, ObjectType,
  COUNT(*) AS TotalRentals,
  SUM(Price) AS TotalIncome
  RANK() OVER(ORDER BY COUNT(*) DESC) AS RANK_TotalRentals,
  RANK() OVER(ORDER BY SUM(Price) DESC) AS RANK_TotalIncome
FROM Object O, Rentals R
WHERE C.ObjectID = R.ObjectID
GROUP BY ObjectID, ObjectType
```
### 2.2
Show, for each object and month
- the object code
- the object type
- the month
- the total income for the object in the current month (SUM(price))
- the object rank according to its total monthly income, separately for each month

```sql
SELECT ObjectID, ObjectType, Month,
  COUNT(*) AS TotalRentals,
  SUM(Price) AS TotalIncome
  RANK() OVER(PARTITION BY Month
              ORDER BY SUM(Price) DESC) AS RANK_TotalIncome
FROM Object O, Rentals R, Time T
WHERE C.ObjectID = R.ObjectID
  AND T.TimeID = R.TimeID
GROUP BY ObjectID, ObjectType, Month
```
## 3. Household
```
SURFACE(storehouseID, timeID, m2free, m2tot)
PRODUCTS(storehouseID, timeID, typeID, totNumber, totValue)
TIME (timeID, date, month, trimester, 4month-period, semester, year)
TYPES (typeID, type, category)
STOREHOUSES (storehouseID, storehouse, city, province, region)
```
### 3.1
In the first trimester of 2003, regarding the storehouses in Turin, select the total value of the products stored in each storehouse at any given date, and select the average daily total value of the products in each storehouse during the previous week (including the current date).
```sql
SELECT storehouse, date,
  SUM(totValue),
  AVG(SUM(totValue)) OVER(PARTITION BY storehouse
                          ORDER BY date
                          RANGE BETWEEN INTERVAL '6' day PRECEDING AND CURRENT ROW)
FROM PRODUCTS P, TIME T, STOREHOUSES H
WHERE P.storehouseID = H.storehouseID
  AND P.timeID = T.timeID
  AND city = Turin
  AND trimester = 1/2003
GROUP BY storehouseID, storehouse, date
```
### 3.2
In 2004, for each city and date, select the percentage of daily free surface of the storehouses. Give a rank to the results (rank 1 is the lowest percentage).
```sql
SELECT city, date,
  SUM(m2free) / SUM(m2tot) * 100,
  RANK() OVER(ORDER BY SUM(m2free) / SUM(m2tot) ASC)
FROM SURFACE S, TIME T, STOREHOUSES H
WHERE S.storehouseID = H.storehouseID
  AND S.timeID = T.timeID
  AND year = 2004
GROUP BY city, date
```
### 3.3
In the first 6 months of 2004, select the percentage of free surface for each storehouse and date.
```sql
SELECT storehouse, date,
  SUM(m2free) / SUM(m2tot) * 100
FROM SURFACE S, TIME T, STOREHOUSES H
WHERE S.storehouseID = H.storehouseID
  AND S.timeID = T.timeID
  AND semester = 1/2004
GROUP BY storehouseID, storehouse, date
```
### 3.4
In 2003, select the average daily total value of products for each storehouse and month.
```sql
SELECT DISTINCT storehouse, month,
  AVG(SUM(totValue)) OVER(PARTITION BY storehouse, month)
FROM PRODUCTS P, TIME T, STOREHOUSES H
WHERE P.storehouseID = H.storehouseID
  AND P.timeID = T.timeID
  AND year = 2003
GROUP BY storehouseID, storehouse, date, month
```
### 3.5
In 2003, select the average daily total value of products for each region.
```sql
SELECT DISTINCT region,
  AVG(SUM(totValue)) OVER(PARTITION BY region)
FROM PRODUCTS P, TIME T, STOREHOUSES H
WHERE P.storehouseID = H.storehouseID
  AND P.timeID = T.timeID
  AND year = 2003
GROUP BY region, date
```
### 3.6
In 2004, select the average percentage of daily free surface for each month and region.
```sql
SELECT DISTINCT month, region,
  AVG(SUM(m2free) / SUM(m2tot) * 100) OVER(PARTION BY month, region)
FROM SURFACE S, TIME T, STOREHOUSES H
WHERE S.storehouseID = H.storehouseID
  AND S.timeID = T.timeID
  AND year = 2004
GROUP BY date, month, region
```
## 4. HouseSearch
```
Properties(monthID, weekID, roomsID, furnitureID, locationID,
  numProperties, totPrice, totSurface)
Favorites(yearID, seasonID, typeID, roomsID, furnitureID, locationID, surfaceRangeID, priceRangeID,
  numUsers, numProperties)
Week(weekID, week)
Month(monthID, month, 2m-period, trimester, 4m-period, semester, year)
Type(typeID, type)
Rooms(roomsID, numRooms)
Furniture(furnitureID, table, bed, ...)
Location(locationID, district, city, university, province, region, area)
Season(seasonID, season)
Year(yearID, year)
Price_Range(priceRangeID, priceMin, priceMax)
Surface_Range(surfaceRangeID, surfaceMin, surfaceMax)
```
### 4.1
In 2004, including only the properties in cities where universities are present, select the average monthly rent cost for each city and month and the average monthly rent cost since the beginning of the year for each city and month.
```sql
SELECT month, city,
  SUM(totPrice)/SUM(numProperties),
  (SUM(SUM(totPrice))/SUM(SUM(numProperties))) OVER(PARTITION BY city
                                                    ORDER BY month
                                                    ROWS UNBOUNDED PRECEDING)
FROM Properties P, Location L, Month M
WHERE P.monthID = M.monthID
  AND P.locationID = L.locationID
  AND year = 2004
  AND university = TRUE
GROUP BY month, city
```
### 4.2
In September 2004, including only the properties in the province of Turin, select the total number of free properties for each city and week, the ratio of the total number of free properties for each city and week and the total number of free properties for the same week.  
Give a rank according to the total number of free properties (rank 1st the highest number). Sort the data according to the rank.
```sql
SELECT week, city,
  SUM(numProperties),
  SUM(numProperties)/SUM(SUM(numProperties)) OVER(PARTITION BY week),
  RANK() OVER(ORDER BY SUM(numProperties) DESC) AS Position
FROM Properties P, Location L, Month M, Week W
WHERE P.weekID = W.weekID
  AND P.monthID = M.monthID
  AND P.locationID = L.locationID
  AND year = 2004
  AND month = 'September'
  AND province = 'Turin'
GROUP BY week, city
ORDER BY Position
```
### 4.3
In summer 2005, including only the attics with bed, fridge, and table in Rome, for each city area (district) and for each range of the monthly rent price, select the average number of users per each property who have added the property in their favorite list, and select the average number of users per each property of the city area who have added the property in their favorite list. Order the results according to the city area and the average number of users.
```sql
SELECT district, priceMin, priceMax,
  SUM(numUsers)/SUM(numProperties) AS avgInterestedUsers,
  (SUM(SUM(numUsers))/SUM(SUM(numProperties))) OVER(PARTITION BY district)
FROM Favorites F, Location L, Month M, Season S, Furniture Fu, Type T, Price_Range PR
WHERE F.seasonID = S.seasonID
  AND F.monthID = M.monthID
  AND F.locationID = L.locationID
  AND F.furnitureID = Fu.furnitureID
  AND F.typeID = T.typeID
  AND F.priceID = PR.priceID
  AND year = 2005
  AND season = 'summer'
  AND type = 'attic'
  AND city = 'Rome'
  AND bed = TRUE AND fridge = TRUE AND table = TRUE
GROUP BY district, priceMin, priceMax
ORDER BY district, avgInterestedUsers
```
### 4.4
Including only the properties with bed and table in cities where universities are present, select the average monthly rent cost per property for each city, month, and year, select the average monthly rent cost per square meter for each city, month, and year, and select the average monthly rent cost per property of the city since the beginning of the year for each city, month, and year.
```sql
SELECT city, month, year,
  SUM(totPrice)/SUM(numProperties),
  SUM(totPrice)/SUM(totSurface),
  (SUM(SUM(totPrice))/SUM(SUM(numProperties))) OVER(PARTITION BY city, year
                                                    ORDER BY month
                                                    ROWS UNBOUNDED PRECEDING)
FROM Properties P, Location L, Month M, Furniture Fu
WHERE P.monthID = M.monthID
  AND P.locationID = L.locationID
  AND P.furnitureID = Fu.furnitureID
  AND university = TRUE
  AND bed = TRUE AND table = TRUE
GROUP BY city, month, year
```
### 4.5
Including only the properties in Piedmont (region), in September, October, and November 2004, select for each city the average monthly rent cost per property and the average monthly rent cost per property of the province in which the city is located.
```sql
SELECT city
  SUM(totPrice)/SUM(numProperties),
  (SUM(SUM(totPrice))/SUM(SUM(numProperties))) OVER(PARTITION BY province)
FROM Properties P, Location L, Month M
WHERE P.monthID = M.monthID
  AND P.locationID = L.locationID
  AND P.furnitureID = Fu.furnitureID
  AND year = 2004
  AND region = 'Piedmont'
  AND month >= 9 AND month <= 11
GROUP BY city
```
### 4.6
In 2004, including only the properties with bed and table in cities where universities are present, select the average monthly rent cost per property for each city and month, and the average monthly rent cost per square meter for each city and month.
```sql
SELECT city, month,
  SUM(totPrice)/SUM(numProperties),
  SUM(totPrice)/SUM(totSurface)
FROM Properties P, Location L, Month M, Furniture Fu
WHERE P.monthID = M.monthID
  AND P.locationID = L.locationID
  AND P.furnitureID = Fu.furnitureID
  AND year = 2004
  AND university = TRUE
  AND bed = TRUE AND table = TRUE
GROUP BY city, month
```
## 5. Hotel
```
Rooms(Features_id, Time_id, Hotel_id, 
  total, free, reserved, unavailable, income)
Features(Features_id, features, shower, refrigerator, whirpool, satellite_TV)
Time(Time_id, day, month, holiday, day_of_week, year)
Hotel(Hotel_id, hotel_name, category, provice, region, state)
```
### 5.1
In 2005, for each state and month, analyze the portion of rooms which are reserved, free, and unavailable.
```sql
SELECT state, month,
  SUM(reserved)/SUM(total)*100,
  SUM(free)/SUM(total)*100,
  SUM(unavailable)/SUM(total)*100
FROM Rooms R, Hotel H, Time T
WHERE R.Hotel_id = H.Hotel_id
  AND R.Time_id = T.Time_id
  AND year = 2005
GROUP BY state, month
```
### 5.2
In 2005, for each state, analyze the portion of rooms which are reserved. Associate a rank to each state according to the portion of reserved rooms for that state in 2005 with respect to all the rooms for that state. The state with the highest ratio of reserved rooms in 2005 must rank first.
```sql
SELECT state,
  SUM(reserved)/SUM(total)*100,
  RANK() OVER(ORDER BY SUM(reserved)/SUM(total) DESC) AS RankState
FROM Rooms R, Hotel H, Time T
WHERE R.Hotel_id = H.Hotel_id
  AND R.Time_id = T.Time_id
  AND year = 2005
GROUP BY state
ORDER BY RankState ASC
```
### 5.3
In 2005, for each state and month, analyze the income of 4-star hotels and thecumulative income of 4-star hotels.
```sql
SELECT state, month,
  SUM(income),
  SUM(SUM(income)) OVER(PARTITION BY state
                        ORDER BY month
                        ROWS UNBOUNDED PRECEDING)
FROM Rooms R, Hotel H, Time T
WHERE R.Hotel_id = H.Hotel_id
  AND R.Time_id = T.Time_id
  AND year = 2005
  AND category = 4
GROUP BY state, month
```
### 5.4
For each state and year, analyze the total income of public holidays.
```sql
SELECT state, year,
  SUM(income)
FROM Rooms R, Hotel H, Time T
WHERE R.Hotel_id = H.Hotel_id
  AND R.Time_id = T.Time_id
  AND holiday = TRUE
GROUP BY state, year
```
### 5.5
In 2005, for each hotel, analyze the total income of the rooms with satellite TV and whirlpool bath.
```sql
SELECT R.hotel_id, hotel_name,
  SUM(income)
FROM Rooms R, Hotel H, Time T, Features F
WHERE R.Hotel_id = H.Hotel_id
  AND R.Time_id = T.Time_id
  AND R.Feature_id = F.Feature_id
  AND year = 2005
  AND satellite_TV = TRUE AND whirlpool_bath = TRUE
GROUP BY R.hotel_id, hotel_name
```
## 6. PC
```
Assembling(timeIdA, componentId, 
  #components, cost, delivery_cost)
Sales(confId, retId, timeIdS, 
  production_cost, selling_price, #PC_sold, #defective_PC_returned)
TimeAss(timeIdA, m, 2m, 4m, 6m, year)
Component(componentId, code, component_type, memory_capacity*, type*, access_time*, resolution*, supplName, supplier_region, supplier_area)
Configuration(confId, RAM_code, HD_code, CPU_code, graphicBoard_Code, RAM_supplier, HD_supplier, CPU_supplier, graphicBoard_supplier) (or *_code as ids)
Supplier(suppId, nameS)
Retailer(retId, nameR, #shops)
TimeSales(timeIdS, m, 2m, 3m, year)
```
### 6.1
Considering only 2002, for each configuration whose RAM has been bought from the “IntelligenceDevice” supplier, select the percentage of defective products with respect to the total number of sold products.
```sql
SELECT confId,
  SUM(#defective_PC_returned)/SUM(#PC_sold)*100
FROM Sales S, TimeSales TS, Configuration C, Supplier CS
WHERE S.confId = C.confId
  AND S.timeIdS = TS.timeIdS
  AND C.RAM_supplier = CS.suppId
  AND year = 2002
  AND CS.nameS = “IntelligenceDevice”
GROUP BY confId
```
### 6.2
Considering only 2007 and RAM components, for each four-month period select the total cost (component cost + delivery cost) and the cumulative cost since the beginning of the year, separately for each geographical area and RAM type.
```sql
SELECT 4m, component_type, supplier_area,
  SUM(cost)+SUM(delivery_cost),
  SUM(SUM(cost)) OVER(PARTITION BY component_type, supplier_area
                      ORDER BY 4m
                      ROWS UNBOUNDED PRECEDING DESC)
FROM Assembling A, TimeAss TA, Component C
WHERE A.componentId = C.componentId
  AND A.timeIdA = TA.timeIdA
  AND year = 2007
  AND component_type='RAM'
GROUP BY 4m, year, component_type, supplier_area
```
### 6.3
Considering only suppliers from which, in 2003, more than 100 000 components have been bought in total, select for each supplier and each component type the number of pieces bought and the total cost in the second two-month period of 2003.
```sql
SELECT supplName, type,
  SUM(#components),
  SUM(cost)+SUM(delivery_cost)
FROM Assembling A, TimeAss TA
WHERE A.timeIdA = TA.timeIdA
  AND year = 2003
  AND 2m = '2-2003'
  AND supplName IN (SELECT supplName
                    FROM Assembling A2
                    WHERE year = 2003
                    GROUP BY supplName
                    HAVING SUM(#components) > 100 000)
GROUP BY supplName, component_type
```
