# MongoDB query exercises
## 1. Book collection
```python
{_id: ObjectId("5fb29ae15b99900c3fa24292"),
  title: "MongoDb Guide",
  tag: ["mongodb", "guide", "database"],
  n: 100,
  review_score: 4.3,
  price: [{v: 19.99, c: "€", country: "IT"},
          {v: 18.00, c: "£", country: "UK"}],
  author: {_id: 1,
           name: "Mario",
           surname: "Rossi"}
},
{_id: ObjectId("5fb29b175b99900c3fa24293",
  title: "Developing with Python",
  tag: ["python", "guide", "programming"],
  n: 352,
  review_score: 4.6,
  price: [{v: 24.99, c: "€", country: "IT"},
          {v: 19.49, c: "£", country: "UK"} ],
  author: {_id: 2,
           name: "John",
           surname: "Black"}
}, ...
```
#### 1.1 Find all the books with a number of pages greater than 250
```python
db.book.find({n:{$gt: 250}})
```
#### 1.2 Find all the books authored by Mario Rossi
```python
db.book.find({"author.name": "Mario", "author.surname": "Rossi"})
```
#### 1.3 Find all the books with a price less than 20€ for the country Italy (IT).
```python
db.book.find({"price": {$elemMatch: {"v": {$lt: 20},
                                     "country": "IT"}}})
```
#### 1.4 Increase the review score of 0.2 points for all the books with the tag “database”.
```python
db.book.updateMany({"tag": "database"},
                   {$inc: {review_score: 0.2}})
```
#### 1.5 Insert the tag “NoSQL” for all the books with tag “mongodb”.
```python
db.book.updateMany({"tag": "mongodb"},
                   {$addToSet: {tag: "NoSQL"}})
```
#### 1.6 Insert the publisher for all the documents authored by Mario Rossi with the default value {‘name’: ‘Polito’, city:’Turin’}.
```python
db.book.updateMany({"author.name": "Mario", "author.surname": "Rossi"},
                   {$set: {publisher: {name: "Polito", city: "Turin"}}})
```
#### 1.7 Find the maximum, the minum, and the average price of all the books with tag “database”.
```python
db.book.aggregate([{$match: {"tag": "database"}},
                   {$unwind: "price"},
                   {$group: {_id: null,
                             avg: {$avg: "price.v"},
                             min: {$min: "price.v"},
                             max: {$max: "price.v"}}}])
```
#### 1.8 Compute the number of books authored by Mario Rossi.
```python
db.book.count({"author.name": "Mario", "author.surname": "Rossi"})
```
## 2. Property listing
```python
{
	"_id": "10006546",
	"url": "https://www.airbnb.com/rooms/10006546",
	"name": "Ribeira Charming Duplex",
	"description": "Fantastic duplex apartment with . . .",
	"property_type": "House",
	"minimum_nights": 2,
	"maximum_nights": 30,
	"beds": 5,
	"number_of_reviews": 51,
	"amenities": [
    "TV",
    "Wifi",
    "Smoking allowed",
    "Pets allowed",
    "Dryer",
    "Heating"
	],
	"price": 80,
	"host": {
    "host_id": "51399391",
    "name": "Ana&Gonçalo",
    "city": "Porto",
    "country": "Portugal"
	}
}
```
#### 2.1 Select all properties that have wifi and dryer as amenities sorted by increasing price. Display only the name, url, and price.
```python
db.collection.find(
  {"amenities": {$all: ["Wifi", "Dryer"]}},
  {_id: 0, name: 1, price: 1, url: 1}
).sort({price: 1})
```
#### 2.2 For all House type properties located in the city of Turin, set the minimum number of nights to 3. If the minimum number of nights is greater than 3, it should not be updated.
```python
db.collection.updateMany(
  {"property_type": "House",
   "host.city": "Turin",
   "minimum_nights": {$lt: 3}},
  {$set: {"minimum_nights": 3}}
)
```
#### 2.3 Calculate the median price of all House type properties.
```python
db.collection.aggregate([
  {$match: {"property_type": "House"}},
  {$sort: {price: 1}},
  {$group: {
    "_id": null,
    "value": {"$push": "$price"}
  }},
  {$project: {
    _id: 1,
    "median": {$arrayElemAt: ["$value", {$floor: {$multiply: [0.50, {$size: $value"}] }}]}
  }}
])
```
## 3. Exams
```python
{
  "_id": "56d5f7eb604eb380b0d8d8d2",
  "student_id": 0,
  "scores": [
    {"type": "exam", "score": 41.25131199553351},
    {"type": "quiz", "score": 91.7351500084582},
    {"type": "homework", "score": 24.198828271948415},
    {"type": "homework", "score": 79.77471812670814}
  ],
  "class_id": 391
}
```
#### 3.1 For each type of test, calculate the minimum, maximum, and average grades taken by students. Display this information only for the types with more than 50 tests taken.
```python
db.collection.aggregate([
  {$unwind: {path: "$scores}},
  {$group: {
    _id: 'scores.type',
    avg: {$avg: 'scores.score'},
    min: {$min: 'scores.score'},
    max: {$max: 'scores.score'},
    count: {$sum: 1}
  }},
  {$match: {count: {$gt: 50}}}
])
```
#### 3.2 Considering only the sufficient scores (grade equal to or higher than 60) of the exam type, calculate for each class the number of exams taken and the average of the grades obtained.
```python
db.collection.aggregate([
  {$unwind: {path: "$scores}},
  {$match: {
    "scores.type": "exam",
    "scores.score": {$gte: 60}
  }}
  {$group: {
    _id: '$class_id',
    avg: {$avg: 'scores.score'},
    count: {$sum: 1}
  }},
])
```
#### 3.3 Enter the certificate grade of 85 for the student with identifier of 1000 enrolled in grade 150. Also set a flag named "completed".
```python
db.collection.updateOne([
  {"student_id": 1000, class_id: 150},
  {$push: {scores: {"type": "certificate", "grade": 85}},
  {$set: {completed: true}}
])
```
## 4. Restaurants (Lab 5)
Each document of the collection has a structure with the following fields:
```python
{ _id: <ObjectId>,
  name: <string>,         # name of the restaurant
  tag: <list[string]>,    # tags assigned by the users
  orderNeeded: <boolean>, # if the user should reserve
  maxPeople:<int>,        # maximum number of customers
  review:<float>,         # average vote
  cost:<string>,          # classification of the menu price: low, medium and high
  location: {
    type: "Point", 
    coordinates: [<lat>, <long>]
  },
  contact: {
    phone: <string>,      # telephone of the restaurant
    facebook: <string>    # link to the facebook page
  }
}
```
#### 4.1 Find all restaurants whose cost is medium. Show the result is the pretty format.
```python
db.restaurants.find({"cost": "medium"}).pretty()
```
#### 4.2 Select the name and the number of seats (maxPeople) available of all the restaurants whose review is bigger than 4 and cost is medium or low.
```python
db.restaurants.find(
  {"review": {$gt: 4},
   $or: [{"cost": "medium"}, {"cost": "low"}]
  },
  {name: 1, maxPeople: 1}
).pretty()
```
#### 4.3 Select the name, the phone of the restaurants that can contain more than 5 people and:
- whose tag contains "italian" or "japanese" and cost is medium or high OR
- whose tag does not contain neither "italian" nor "japanese", and whose review is higher than 4.5

Remove from the output the field \_id.
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.4 Calculate the average review of all restaurants.
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.5 Count the number of restaurants whose review is higher than 4.5 and can contain more than 5 people.
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.6 Find the restaurant in the collection which is nearest to the point [45.0644, 7.6598]. Hint: remember to create the geospatial index.
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.7 Find how many restaurants in the collection are within 500 meters from the point [45.0623, 7.6627].
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.8 Add the tag “pizza” to all the restaurants that contain the tag “italian”. If the tag “pizza” is already present, you should not insert it.
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.9 Decrease the review score of 0.2 for all the restaurants with the tag "fastfood".
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.10 For only the restaurants with a review higher than 3, find the tags which appear more than 1 time. For each tag, show how many documents include it.
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.11 For each cost category, compute the minimum review rate, the maximum review rate, the average review rate and the number of restaurants. Sort the result in descending order according to the number of restaurants in each cost category.
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
#### 4.12 Find the median value of maxPeople attribute.
```python
db.restaurants.find({"cost":"medium"}).pretty()
```
