# MongoDB design pattern exercises
## 1. Books and publishers
We want to design a MongoDB database to store the following data about books and publishers.
- Each publisher is characterized by the name, the year of foundation and the country. 
A list of contact methods is available. The possibile contact methods are: email, telephone, website. 
At most one contact for each method can be recorded. Additional methods might be added in the future.
- Each book is identified by the ISBN code and characterized by the title, the subtitle, the number of pages, 
the language and the publication date. Each book is published by only one publisher. 
A publisher distributes several books.
- The design should be optimized to display for each book all its information and the publisher's name.

Indicate for each collection in the database the document structure and the strategies used for modelling.
```python
publisher{
  _id: <ObjectId>,
  name: <string>,
  year: <date>,
  country: <string>,
  contacts: {email: <string>,
             tel: <string>,
             website: <string>}
}
# polymorphic pattern to track only the contact methods that are available

book{
  _id: <string>, #ISBN
  title: <string>,
  subtitle: <string>,
  n: <int>,
  language: <string>,
  pub_date: <date>,
  country: <string>,
  publisher: {_id: <ObjectId>,
              name: <string>}
}
# Extended reference pattern to display the name of the publisher. 
# The application can access the book collection to show both the book information and the name of the publisher.
# In this way we avoid a lookup to the publisher collection.
```
## 2. Blog website
We want to design a MongoDB database for the management of a blog.
- Each user subscribed to the blog is identified by a username. 
We want to keep track of user contact methods, such as email, phone, Twitter account, etc. 
The contact methods tracked by the platform may change over the time. 
A user might have more contacts of the same method. 
Each contact method can have a status, e.g., “active”, “inactive”, “confirmed”, etc.
- Blog posts are characterized by the title, the description and the author (i.e., username). 
The posts receive comments from users. Each comment is characterized by the text, the user who published it and its timestamp.
- The application should display for each post only the top 10 most recent comments.
The user profile should also include the total number of posts and comments written by the user.

Indicate for each collection in the database the document structure and the strategies used for modelling.
```python
user{
  _id: <string>, #username
  contacts: [k: <string>,   #contact method
             v: <string>,   #contact value
             s: <string>],  #status
  nPosts: <int>,
  nComments: <int>
}
# Attribute pattern to track the contact methods.
# The embedded documents in the contact fields consists of the k field which represents the contact method, 
# the v field providing the corresponding value, and the s field for the status.
# Computed pattern to store the number of posts and comments published by the user.

post{
  _id: <ObjectId>,
  title: <string>,
  description: <string>,
  authon: <string>, #username
  comments: [
            {_id: <ObjectId>
             user: <string>,
             text: <string>,
             timestamp: <datetime>}, 
             ...],  #most recent comments
}
# Subset pattern to track only the most recent comments.

comment{
  _id: <ObjectId>
  user: <string>,
  text: <string>,
  timestamp: <datetime>
}
```
## 3. Breadcrumbs
We want to design the database for the management of a sitemap.
- Each page of the webisite is characterized by the relative path, the title, the body, the creation timestamp, the last update timestamp and a list of tags.
- The pages are organized in a tree strcture. The home page is the root. All other pages are in sub paths from the root.
- The application should build the breadcrumbs code for each page. The breadcrumbs are an ordered list of links containing all the ancestors of the current page back to the home page, i.e., the root.

Indicate for each collection in the database the document structure and the strategies used for modelling.
```python
page{
  _id: <ObjectId>,
  path: <string>,
  title: <string>,
  body: <string>,
  creation: <datetime>,
  update: <datetime>,
  tags: [ <string> ],
  parent: {_id: <ObjectId>,
           path: <string>}
  ancestors: [
              {_id: <ObjectId>,
               path: <string>},
              {_id: <ObjectId>,
               path: <string>},
              ... ]
}
# Tree pattern to track the ancestors. The parent field is added to use, if necessary, the $lookup and $graphlookup functions.
# For each ancestor, the extended reference pattern is exploited to include the path in the embedded document and avoid join operations.
```
## 4. Parcel delivery
Design a MongoDB database to manage the parcel delivery according to the following requirements.
- Customers of the parcel delivery service are citizens identified by their social security number. They can be senders or recipients of delivered parcels. They are characterized by their name, surname, email address, a telephone number, and by different addresses, one for each type (e.g., one billing address, one home address, one work address, etc.). Each address consists of street name, street number, postal code, city, province, and country.
- Parcels are characterized by a unique barcode and their physical dimensions (specifically: width, height, depth, and weight). All widths, heights, and depths are always expressed in meters. All weights are always expressed in kilograms.
- The recipient and the sender information required to deliver each parcel must be always available when accessing the data of a parcel. Recipient and sender information required to deliver a parcel consists of full name, street name, street number, postal code, city, province, and country.
- The parcel warehouse is divided into different areas. Each area is identified by a unique code, e.g., 'area_51' and consists of different lines. Each line is identified by unique code, e.g., 'line_12', and hosts several racks. Each rack is identified by unique code, e.g., 'rack_33', and is made up of shelves. Each parcel is placed on a specific shelf of the warehouse, identified by a unique code, e.g., 'shelf_99'. The database is required to track the location of each parcel within the warehouse.
- Given a parcel, the database must be designed to efficiently provide its full location, from the shelf, up to the area, through rack and line.
- Given a customer, the database must be designed to efficiently provide all her/his parcels as a sender, and all his/her parcels as a recipient.

Indicate for each collection in the database the document structure and the strategies used for modelling.
```python
customer{
  _id: <string>, #SSN
  name: <string>,
  surname: <string>,
  email: <string>,
  tel: <string>,
  addresses: {
              'home': {
                street_name: <string>,
                street_num: <string>,
                postal_code: <int>,
                city: <string>,
                province: <string>,
                country: <string>
              },
              'billing': {
                street_name: <string>,
                street_num: <string>,
                postal_code: <int>,
                city: <string>,
                province: <string>,
                country: <string>
              },
              'work': {...}
            }
}
# Attribute pattern (optional) for the addresses attribute.

parcel{
  _id: <string>, #barcode
  dimensions: {
    width: <number>,
    height: <number>,
    depth: <number>,
    weight: <number>
  },
  recipient: {
    _id: <string>,
    street: <string>,
    number: <string>,
    zip_code: <string>,
    city: <string>,
    province: <string>
  },
  sender: {
    _id: <string>,
    street: <string>,
    number: <string>,
    zip_code: <string>,
    city: <string>,
    province: <string>
  },
  father_position: <string>,
  locations: [ <string> ]
}
# Extended reference pattern for recipient and sender address information. The recipient and sender _ids are required to look up all parcels of a given customer.
# Tree pattern for the position. The full list of tree- pattern ancestors is required. The parent ancestor of the tree-pattern is optional.
```
## 5. Museum exhibitions
We want to design the database to manage museum exhibitions according to the following requirements.
- Museums are characterized by their name, address, a telephone number, and a website (if available). The address consists of geographical coordinates, street name and number, postal code, and city.
- The items exhibited in the museums are identified by a progressive number and characterized by a title, a description and the list of author names. The items are categorized as either archaeological finds, or paintings, or sculptures.
- The database must record all the main features of each item, such as its dimensions (i.e., width, height, weight, etc.). Each feature has at least a name and a value, and possibly a unit of measure. For instance, the main material is a feature of an archaeological find, the geometrical sizes are features of a painting. For each item, the museum to which it belongs must be recorded, with the museum name frequently accessed together with the item itself.
- Several exhibitions are hosted in each museum. The exhibition is characterized by a title, a description, the list of curator names. You must record all the items associated with each exhibition; they can be in the order of hundreds. An item can be part of different exhibitions. Moreover, each exhibition can be hosted by several museums in different periods. You must record the start and end dates of each exhibition in each museum.
- Given an item, the database must be designed to efficiently provide the name of the museum that owns it.
- Given an exhibition, the database must be designed to efficiently provide the name of the museum and the geographical coordinates where it has been hosted.
- Furthermore, given an exhibition, the list of items included in the exhibition and their number must be efficiently returned.

Indicate for each collection in the database the document structure and the strategies used for modelling.
```python
museums{
  _id: <ObjectId>,
  name: <string>,
  address: {
              street_name: <string>,
              street_num: <string>,
              postal_code: <int>,
              city: <string>,
              province: <string>,
              geo_ref: {
                type: <string>,
                coordinates: [ <number> ]
              }
            },
  phone: <string>,
  website: <string>
}
# Polymorphic pattern to track the museum information in the museum collection.

item{
  _id: <number>,
  title: <string>,
  description: <string>,
  authors: [ <string> ],
  category: <string>,
  features: [ {k: <string>, v: <string>, u: <string>} ],
  museums: {
              _id: <ObjectId>,
              name: <string>
           }
}
# Attribute pattern (with polymorphic pattern) to track the features of each item.
# Extended reference pattern to track the museum information associated with each item.

exhibition{
  _id: <ObjectId>,
  title: <string>,
  description: <string>,
  curators: [ <string> ],
  events: [
            {start: <date>,
             end: <date>,
             museum: {
               _id: <ObjectId>,
               name: <string>,
               geo_ref: {
                  type: <string>,
                  coordinates: [ <number> ]
                }
             }
            }
          ],
  items: [ <number> ]
}
# Bucket pattern with extended reference pattern to track when an exhibition is hosted in a museum.
