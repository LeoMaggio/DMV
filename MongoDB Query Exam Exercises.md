# MongoDB Query Exam Exercises
## February 01, 2021
#### 1. Given the following document structure:
```python
{
  "address":
    {"building": "768",
    "coord": [-73.9685872,40.7679509],
    "street": "Madison Avenue",
    "zipcode": "10065",
    "borough": "Manhattan",
    "city": "New York"},
  "sold_items": ["Smartphones", "PC", "TV"],
  "reviews": [
    {"date": {date: {"2019-11-05"}}, "score": 10, "description": "Lorem ipsum"},
    {"date": {date: {"2020-02-21"}}, "score": 8, "description": "Lorem ipsum"}
  ],
  "name": "Elettronic-store"
}
```
Select all the shops located in Rome that sell smartphones or TV and received at least one review with a score greater than 8. Show only the name, the street and the building.
```python
db.shops.find({
  sold_items: {$in: ["Electronics", "Home"]},
  "address.city": "Rome",
  "reviews.score: {$gt: 8}
  },
  {_id: 0, name: 1, "address.street": 1, "address.building": 1}
)
```
#### 2. Given the following document structure:
```python
{
  "name": "Electrostore",
  "address":
    {"building": "A1",
    "street": "via Torino",
    "zipcode": "12345",
    "borough": "Campidoglio",
    "city": "Rome"},
  "sold_items": ["Smartphone", "PC", "TV"],
  "reviews": [
    {"date": "2019-11-05", "score": 10, "description": "Lorem ipsum"},
    {"date": "2020-02-21", "score": 7, "description": "Lorem ipsum"}
  ]
}
```
For each city, compute the average and the maximum review score. Show only the first 10 cities with the highest number of reviews.
```python
db.collection.aggregate([
  {$unwind: "$reviews"},
  {$group: {
    _id: "$address.city",
    'countReviews': {$sum: 1},
    'maxReviewScore': {$max: '$reviews.score'},
    'avgReviewScore': {$avg: '$reviews.score'}
  }},
  {$sort: {countReviews: -1}},
  {$limit: 10}
])
```
## February 15, 2021
Given the following document structure, representing the measurements received bysensors, where each document collects the measures received in one day:
```python
{"_id": ObjectId("5553a998e4b02cf7151190b8"),
  "start": Date("2021-02-01T00:00:00.000Z"),
  "end": Date("2021-02-01T23:00:00.000Z"),
  "sensor": {
    "_id": 1000,
    "position": {"type": "Point", "coordinates": [-47.9,47.6]},
    "elevation": 200,
    "city": "Turin",
    “country”: “Italy”
  },
  "temperature": [
    {ts: Date("2021-02-01T00:00:00.000Z"), value: 12},
    {ts: Date("2021-02-01T01:00:00.000Z"), value: 11},
    ...
    {ts: Date("2021-02-01T23:00:00.000Z"), value: 9}
  ],
  nTemp: 24,    # total number of elements in the temperature list
  sumTemp: 372  # sum of the values of all elements in the temperature list
}
```
Update the document of the sensor with "\_id" equal to 1000 by adding a new "temperature" measurement with "value" 16 received at the timestamp "ts" 2021-02-02T01:10:00.000Z. Also concurrently update the corresponding statistics (i.e., "nTemp" and "sumTemp"). Suppose that the document with "start" attribute equal to "2021-02-02" exists.
```python
db.measures.updateOne(
  {'sensor._id': 1000,
   'start': new Date("2021-02-02")},
  {$inc: {nTemp: 1, sumTemp:16},
   $push: {temperature: {ts: new Date("2021-02-20 10:00"), value: 16}}}
)
```
Considering the sensor located in Italy and the measures received in the month of January 2021, show the sensor id, sensor city and the date in which the average measure of the sensor was greater than or equal to 15.
```python
db.measures.aggregate([
  {$match: {
    "sensor.country": "Italy",
    "start": {$gte: new Date("2021-01-01")},
    "end": {$lte: new Date("2021-01-31")},
    {$addField: {avg: {$divide: ["$tot", "$n"]}}
  },
  {$match: {avg: {$gte: 15}}},
  {$project: {"sensor._id": 1, "sensor.city": 1, start: 1}}
])
```