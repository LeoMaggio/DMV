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
