# Extended SQL Additional Exercises
## 1. Car Rentals
```
CarRental(ArrivalID, DepartureID, VehicleID, ClientID, RentalTimeID, paymentMode, num_rental, num_kilometers, revenue)
LOCATION(LocationID, city, province, region)
VEHICLE(VehicleID, model, type, brand)
OPTIONS(OptionsID, name)
VEHICLEOPTIONS(VehicleID, OptionsID)
RENTALTIME(RentalTimeID, date, month, monthOfYear, 2m, 3m, 6m, year, dayOfWeek, workingDay)
CLIENTPROFILE(ClientID, clientType, clientProvince, clientRegion)
```
#### 1.1 Consider the rentals payed with credit card. Separately for each month, client type, province of the client’s domicile, analyze:
- the average number of kilometers for rental, 
- the cumulative number of kilometers from the beginning of the year, 
- the average daily kilometers.
```sql
SELECT month, clientType, clientProvince, year
  SUM(num_kilometers)/SUM(num_rental),
  SUM(SUM(num_kilometers)) OVER(PARTITION BY clientType, clientProvince, year
                                ORDER BY month
                                ROWS UNBOUNDED PRECEDING)
  SUM(num_kilometers)/COUNT(DISTINCT date)
FROM CarRental R, RENTALTIME T, CLIENTPROFILE C
WHERE R.ClientID = C.ClientID
  AND R.RentalTimeID = T.RentalTimeID
  AND paymentMode = 'credit card'
GROUP BY month, clientType, clientProvince, year
```
#### 1.2 Consider the rentals of "truck" vehicle type. Separately for each rental start month and for each departure province, analyze:
- the average revenue per kilometer, 
- the percentage of revenue over the total revenue of the year, 
- the percentage of revenue over the total revenue for the corresponding departure region
```sql
SELECT month, province, year, region
  SUM(revenue)/SUM(num_kilometers),
  SUM(revenue)100*/SUM(SUM(revenue)) OVER(PARTITION BY province, year)
  SUM(revenue)100*/SUM(SUM(revenue)) OVER(PARTITION BY region, month)
FROM CarRental R, RENTALTIME T, VEHICLE V, LOCATION L
WHERE R.VehicleID = V.VehicleID
  AND R.RentalTimeID = T.RentalTimeID
  AND R.DepartureID = L.DepartureID
  AND type = 'truck'
GROUP BY month, province, year, region
```
