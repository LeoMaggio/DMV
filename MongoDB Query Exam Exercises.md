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
