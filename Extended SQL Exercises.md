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
SELECT Province,
  Region,
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
SELECT Province,
  Region,
  Month,
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
SELECT Region,
  Month,
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
SELECT ObjectID,
  ObjectType,
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
SELECT ObjectID,
  ObjectType,
  Month,
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
**FACTS**
```
SURFACE (storehouseID, timeID, m2free, m2tot)
PRODUCTS (storehouseID, timeID, typeID, totNumber, totValue)
```
**DIMENSIONS**
```
TIME (timeID, date, month, trimester, 4month-period, semester, year)
TYPES (typeID, type, category)
STOREHOUSES (storehouseID, storehouse, city, province, region)
```
### 3.1
In the first trimester of 2003, regarding the storehouses in Turin, select the total value of the products stored in each storehouse at any given date, and select the average daily total value of the products in each storehouse during the previous week (including the current date).
```sql
SELECT storehouse,
  date,
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
SELECT city,
  date,
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
SELECT storehouse,
  date,
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
SELECT DISTINCT storehouse,
  month,
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
SELECT DISTINCT month,
  region,
  AVG(SUM(m2free) / SUM(m2tot) * 100) OVER(PARTION BY month, region)
FROM SURFACE S, TIME T, STOREHOUSES H
WHERE S.storehouseID = H.storehouseID
  AND S.timeID = T.timeID
  AND year = 2004
GROUP BY date, month, region
```
## 4. HouseSearch
**FACTS**
```
Properties(monthID, weekID, roomsID, furnitureID, locationID,
  numProperties, totPrice, totSurface)
Favorites(yearID, seasonID, typeID, roomsID, furnitureID, locationID, surfaceRangeID, priceRangeID,
  numUsers, numProperties)
```
**DIMENSIONS**
```
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
SELECT month,
  city,
  SUM(totPrice)/SUM(numProperties),
  (SUM(SUM(totPrice))/SUM(SUM(numProperties))) OVER(PARTITION BY City
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
