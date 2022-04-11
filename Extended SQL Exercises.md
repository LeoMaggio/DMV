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
