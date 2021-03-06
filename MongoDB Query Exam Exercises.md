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
    "position": {"type": "Point", "coordinates": [-47.9, 47.6]},
    "elevation": 200,
    "city": "Turin",
    ???country???: ???Italy???
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
## June 17, 2021
The following document structure represents cameras sold by an e-commerce. Each document collects the aggregated metrics of one day.
```python
{ "_id": "nikon_d3500",
  "model": "D3500",
  "brand": {
    "name": "Nikon",
    "url": "https://www.nikon.it/"
  }
  "releaseDate": Date("2018-08-28T00:00:00.000Z"),
  "category": "DSRL",
  "price": 435,
  "specs": {
    "resolution": 24,
    "technology": "APS-C CMOS",
    "min_ISO": 100,
    "max_ISO": 25600,
    "weight": 365,
    "viewfinder": "optical",
    "video_resolution": "1920 x 1080"
  },
  "scores": {
    "overall": 57,
    "image_quality": 48,
    "versatility": 62,
    "comfort": 85,
    "speed": 41
  }
}
```
Write a MongoDB query to display only the model, the price, and the brand name of cameras released in 2021, belonging to the "laser" category, and whose overall score is in the 70-90 range.
```python
db.cameras.find(
  {
    category: 'laser',
    releaseDate: {
      $gte: new Date('2021-01-01'),
      $lt: new Date('2022-01-01')
    },
    'scores.overall': {
      $gte: 70,
      $lte: 90
    }
  },
  {model: 1, "brand.name": 1, price: 1, _id: 0}
)
```
Considering only cameras released since 2015, for each release year and for each category, select the median overall score. Use the operator `$year` to extract the year from the date, e.g., `$year: "$releaseDate"`.
```python
db.measures.aggregate([
  {$match: {releaseDate: {$gte: new Date('2015-01-01')}}},
  {$sort: {'$scores.overall': 1}},
  {$group: {
    '_id': {
      'cat': '$category',
      'y': { $year: "$releaseDate" }
    }
  },
  {$project: {
    _id: 1,
    "median": {
      $arrayElemAt: ["$value", {
        $floor: {$multiply: [0.50, {$size: "$value"}]}
      ]}
    }
  }
])
```
## September 01, 2021
The following document structure represents online courses.
```python
{
  "_id": ObjectId(???xyz???),
  "title": "Python 3.9",
  "teacher": {
    "name": "John",
    "surname": "Doe",
    "webiste": "https://www.doe.com/",
    "nation": "USA"
  },
  "published": Date("2019-02-13T00:00:00.000Z"),
  "category": "Computer Science",
  "tags": ["Python", "Coding"],
  "price": 99,
  "avg_score": 4.8,
  "number_reviews": 47,
  "enrolled_students": 1234,
  "details: {
    "hour_length": 12,
    "number_of_lessons": 38,
    "final_test": false
  }
}
```
Write a MongoDB query to display only the title, the category, and the price of courses containing the tag "Databases", published in 2019, and whose length is less than 10 hours.
```python
db.cameras.find(
  {
    tag: 'Databases',
    published: {
      $gte: new Date('2019-01-01'),
      $lt: new Date('2020-01-01')
    },
    'details.hour_length': {
      $lt: 10
    }
  },
  {"title": 1, "category": 1, "price": 1, "_id": 0}
)
```
Considering only courses in the category Computer Science published in the year 2020, for each tag, select the average price and the maximum number of enrolled students.
```python
db.measures.aggregate([
  {$match: {
    "category": "Computer Science",
    "published": {
      $gte: new Date("2020-01-01"), 
      $lte: new Date("2021-01-01")
    },
  },
  {$unwind: "$tags"},
  {$group: {
    "_id": "$tags",
    "avg_price": {"$avg": "$price"},
    "max_students": {"$max": "$enrolled_students"}
  }}
])
```
## January 28, 2021
The following document structure represents a document of a set of events measuredby a sensor. Each document collects all the measurements of a sensor for a given date.
```python
{ "_id": ObjectId("xyz"),
  "sensor": {
    "id": 1,
    "location": {
      "building": "A"
      "floor": 1,
      "type": "bedroom"
    }
  },
  "start???: Date("2022-01-15T00:00:00.000Z"),
  "measures" : [
    {"ts": Date("2022-01-15T01:59:59.000Z"), "temperature": 21},
    {"ts": Date("2022-01-15T23:59:59.000Z"), "temperature": 21.5}
  ],
  "n": 2,
  "sum_temp": 42.5
}
```
Write a MongoDB query to insert a new measure for the sensor 5 acquired on 2021-12-01 at 08:00 with a temperature equal to 19.
- The document to be updated is related to the sensor 5 and to the set of measures of the day 2021-12-01 (start attribute).
- Increase also the statistics (i.e., attributes "n" and "sum_temp") stored in the document, which describe the number of measurements and the sum of the temperatures.
```python
db.timeserie.updateOne(
{ 'sensor.id': 5,
  'start': new Date(2021-12-01T00:00:00.000Z) },
{ '$push': {'measures': {ts: new Date("2021-12-01T08:00:00.000Z"), ???temperature???: 19}},
  '$inc': {'n':1, ???sum_temp???: 19} }
)
```
Considering only measurements acquired in June 2021 by sensors located at floor 2, foreach building, select the average and the maximum temperature.
```python
db.timeserie.aggregate([
  {$match: {
    "sensor.location.floor": 2,
    "start": {$gte: new Date("2021-06-01"), $lte: new Date("2021-06-30")}
  }},
  {$unwind: "$measures"},
  {$group: {
    '_id': '$sensor.location.building',
    'avg_temp': {'$avg': '$measures.temperature'},
    'max_temp': {'$max': '$measures.temperature'}
   }}
])
```
