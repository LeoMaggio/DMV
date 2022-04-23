# NoSQL Database Design exam exercises
## January 31, 2020
Design a document-based NoSQL database for storing the following data of a company.
- Employees, described by their
  - name,
  - surname,
  - list of education certificates (e.g., "Master Degree", "PhD Degree", etc.),
  - list of addresses, each one consisting of city, postal code, street name, and street number
  - identification number (a unique integer identifying each employee).
- Projects, described by their
  - name (two different projects cannot have the same name),
  - time period, consisting of start date, end date, duration in days, duration in months,
  - list of topics (e.g., "mobile app", "web service", "machine learning", etc.).

The number of projects is virtually unbounded, as every day new projects are created. Employees are assigned to many projects, and potentially, the list of historical projects an employee was assigned to can grow indefinitely over time. The company has a limited number of employees, and each project has a limited number of employees assigned.

Consider the following access pattern to the data.
- The employee data is presented on a dashboard where the employee identification number, name, surname, list of education certificates, and list of addresses are always displayed.
- The project data is presented on a dashboard where the project name, start date, end date, duration in days, duration in months, list of topics, and list of assigned employees, with their identification number, name, and surname, are always displayed.

Provide below a relevant sample document for each collection you design to address the described context, e.g., a sample employee document and a sample project document with sample values for all the attributes you expect to store inside each document.
```python
employee{
  _id: <number>,                      # 123456
  name: <string>,                     # 'Leonardo'
  surname: <string>,                  # 'Maggio'
  education: [ <string> ],            # ['BSc', 'MSc', ...]
  addresses: [
              {street_name: <string>, # 'Via Giovanni da Verazzano'
               street_num: <string>,  # 59
               postal_code: <number>, # 10129
               city: <string>},       # 'Turin'
              {street_name: <string>, # 'Via Alessandro Manzoni'
               street_num: <number>,  # 7
               postal_code: <number>, # 73019
               city: <string>}, ...   # 'Trepuzzi' 
  ]
}
project{
 name: <string>,                      # 'Project97'
 timeperiod: {
              start: <date>,          # 2022-04-19
              end: <date>,            # 2022-04-28
              days: <number>,         # 9
              months: <number>        # 0
 }
 topics: [ <string> ]                 # ['data base systems', 'machine learning', ...]
 employees: [
              {_id: <string>,         # 123456
               name: <string>,        # 'Leonardo'
               surname: <string>},    # 'Maggio'
              {_id: <string>,         # 123457
               name: <string>,        # 'Mario'
               surname: <string>},    # 'Rossi'
  ]
}
```
## February 14, 2020
Design a document-based NoSQL database for storing the following data of a gym.
- Gym courses, described by
  - course name (unique),
  - list of course categories, each consisting of a name and an associated colour,
  - course instructors,
  - year,
  - weekly schedule of the lessons, consisting of weekday, start time, and end time for each lesson.
- Instructors, described by their
  - name and surname,
  - date of birth
  - list of certifications, each consisting of a name and the date of achievement
  - list of the courses.

Most courses have only one instructor, and no course has a large number of instructors. The number of courses can grow indefinitely, as new courses are created every year. The number of instructors is limited, however, the number of courses an instructor has been teaching can grow indefinitely. The colour of the course category must be saved as a sequence of three numbers in the range 0-255 (RGB, Red Green Blue standard, e.g., white colour is 255, 255, 255).

Consider the following access pattern to the data.
- The gym courses of the current year are presented on a website where their name, categories (each coloured accordingly), name and surname of the instructors, and the weekly schedule are to be displayed at every page view.
- The instructor information is presented on an internal administration software, where the instructor name, surname, date of birth, and list of certifications are presented for each instructor, together with the number of courses taught, separately for each year (e.g., instructor "A": year 2019, 23 courses; year 2018, 45 courses; etc.).

Provide below a relevant sample document for each collection you design to address the described context, e.g., a sample course document and a sample instructor document with sample values for all the attributes you expect to store inside each document, to minimize the access time to the data.
```python
course{
  name: <string>,             # 'Powerlifting'
  categories: [
    {name: <string>,          # 'Strength training'
     colour: [ <number> ]},   # [255, 255, 255]
    {name: <string>,          # Fitness
     colour: [ <number> ]},   # [0, 0, 0]
    ...
  ]
  instructors: [
    {name: <string>,          # 'Leonardo'
     surname: <string>},      # 'Maggio'
    {name: <string>,          # 'Lorenzo'
     surname: <string>},      # 'Bianco'
    ...
  ],
  year: <number>              # 2022
  schedule: [
    {weekday: <string>,       # 'Monday'
     start: <datetime>,       # 10:30
     finish: <datetime>},     # 11:30
    {weekday: <string>,       # 'Tuesday'
     start: <datetime>,       # 10:30
     finish: <datetime>},     # 11:30
    ...
  ]
}
instructor{
  _id: <ObjectId>,            # 'ABC123'
  name: <string>,             # 'Leonardo'
  surname: <string>,          # 'Maggio'
  dateOfBirth: <datetime>,    # 1997-12-01
  certifications: [
    {name: <string>,          # 'Strength training Level 1'
     date: <datetime>},       # 2022-04-28
    {name: <string>,          # 'Fitness training Level 1'
     date: <datetime>},       # 2022-04-28
    ...
  ],
  courses: [
    {year: <number>,          # 2020
     count: <number>},        # 52
    {year: <number>,          # 2021
     count: <number>},        # 52
  ]
}
```
## June 18, 2020
Design a document-based NoSQL database for storing the following data describing restaurant dishes and deliveries, and specifically optimizing the described access patterns.

A restaurant is described by the following data.
- A unique identification string, starting with “R” and followed by a unique number.
- A name, as indicated on its public sign.
- An address, consisting of street name (e.g., “Via Po”), house number (e.g., “1A”), city name, postal code (e.g., 10100), and country name.

A dish is described by the following data.
- A unique identification string, starting with “D” and followed by a unique number.
- A name, as indicated on the menu (e.g., “Super-size hot dog”).
- Different prices, such the regular price, the promotional price, the discounted price, etc.
- An approximate weight value in kg.
- A list of the main ingredients, with each ingredient being a text string, such as “bread”, “wurstel”, etc.
- A list of filenames of the pictures of the dish.

Each restaurant belongs to one or more categories, such as pizza, sea food, or street food. The number of categories a restaurant belongs to is low and limited. It is often required to filter restaurants based on their categories.

Each restaurant has a list of dishes in its menu. Dishes are always accessed by their identification number. Each dish might be in the menu of more than one restaurant. The number of dishes for each restaurant can be large, but it is finite.

The access pattern to the above-described data typically consists of (i) a restaurant search, then, after selecting a restaurant, (ii) its dishes are accessed in a second interaction with a menu preview of the dish name and its regular price. Finally, (iii) selecting a dish from the preview menu presents the full data of the selected dish to the customer.

Customers of the restaurants can ask to deliver some dishes at their address. The database must record all deliveries of all dishes for all restaurants. The number of deliveries can grow indefinitely.

A delivery is described by the following data and is typically accessed by the delivery courier who needs to know the following information in their mobile app for managing the deliveries.
- A unique identification timestamp automatically generated by the database.
- The customer id (e.g., “c123456”), the customer full name, the customer phone number, and the customer address (in the same format as the restaurant address).
- The list of dishes, which are rarely accessed by the delivery courier.
- The restaurant id, name, and address.

Provide below a relevant sample document for each collection you design to address the described context.
- E.g., if you design the database with four collections, a sample document for each collection must be provided, writing explicitly the collection name before each sample document.
- Sample documents must provide values for all the attributes you expect to store inside them.
- When lists are used, provide at least 2 sample items in each list.
- Please use a new line for each new attribute of the document, also for sub-documents.
- Please use pretty indentation to make the document human readable.
```python
restaurant{
  _id: 'R123456',
  name: 'Piola da Cianci',
  address: {
    street_name: 'Largo IV Marzo',
    number: '9B',
    city: 'Turin',
    zip_code: 10122,
    country: 'Italy'
  },
  categories: [ 'local cuisine', 'tavern', ...] 
  dishes: [
    {_id: 'D123456', name: 'Vitello Tonnato', price_regular: 12},
    {_id: 'D123457', name: 'Tartare di Fassona', price_regular: 15},
    ...
  ]
}
dish{
  _id: 'D123456',
  name: 'Vitello tonnato',
  prices: {
    regular: 12,
    prom: 10,
    discounted: 8
  },
  weight: 0.5,
  ingredients: [
    'cow meat',
    'tuna',
    ...
  ],
  pictures: [
    'images/vitello_tonnato1.jpg',
    'images/vitello_tonnato2.jpg',
    ...
  ]
}
delivery{
  _id: '202204211338',
  customer: {
    _id: 'C123456',
    full_name: 'Leonardo Maggio',
    phone: '+393487109579',
    address: {
      street_name: 'Via Giovanni da Verazzano',
      number: '59',
      city: 'Turin',
      zip_code: 10129,
      country: 'Italy'
    }
  },
  dishes: [ 'D123456', 'D123457', ... ],
  restaurant: {
    _id: 'R123456',
    name: 'Piola da Cianci',
    address: {
      street_name: 'Largo IV Marzo',
      number: '9B',
      city: 'Turin',
      zip_code: 10122,
      country: 'Italy'
    }
  }
}
```
## June 17, 2021
Design a MongoDB database to manage online courses according to the following requirements.
- Teachers are characterized by their name, surname, email, and list of subjects they can teach (e.g., maths, electronics, etc.). Each teacher can have one or more online profiles on different platforms (e.g., Facebook, LinkedIn, Wikipedia, etc.). For each online profile, if available, the database tracks the corresponding URL of the profile (e.g., https://en.wikipedia.org/wiki/Ranjitsinh_Disale). Note that for each teacher and each platform, at most one profile can exist. A teacher can teach different courses.
- The courses are characterized by a name, a syllabus, a list of keywords, and the teacher. Each course has several editions. For each edition, the start date, the end date, and the number of enrolled students are known.
- Given a course, the database must be designed to efficiently provide the name, the surname and the email of its teacher.
- Furthermore, given a course, the number of editions and the average number of enrolled students in each edition must be efficiently returned.
- Teachers are typically retrieved by subject (e.g., all those teaching maths), and by online profile platform (e.g., all those having a wikipedia page).

Write a sample document for each collection of the database.
```python
teacher{
  _id: ObjectId(),
  name: <string>,
  surname: <string>,
  email: <string>,
  subjects : [ <string> ],
  profile: {
    facebook: <url>,
    linkedin: <url>,
    ...
  },
}
# Polymorphic pattern to track the online profile information in the Teacher collection.
course: {
  _id: ObjectId(),
  name: <string>,
  syllabus: <string>,
  keywords: [ <string> ],
  teacher: {
    _id: ObjectId(),
    name: <string>,
    surname: <string>,
    email: <string>
  },
  editions [{
    start: <date>,
    end: <date>,
    n_students: <number>
  }],
  n_editions: <number>,
  tot_students: <number>
}
# Extended reference pattern to track the teacher information associated with each course.
# Bucket pattern to track when a course is provided.
# Computed pattern for average students on each edition.
```
## September 01, 2021
Design a MongoDB database to store reviews of hotels from a website according to the following requirements.
- The data to be displayed on the review website for each hotel include the hotel name, the number of stars, and the list of provided services, e.g., free wifi, baby parking, pet allowed, etc.
- For each hotel, the venue information and its top 10 reviews must be always shown.
- The venue information consists of the address, the city, and the country. Furthermore, the official website address might be included in the venue information.
- Each review consists of a timestamp, a score (e.g., 4.5), the nickname of its author, the number of "likes", and a textual description. Each review is related to one specific hotel.
- Given a hotel, the database must be designed to efficiently provide all the data describing the hotel, its top 10 reviews (those having the highest numbers of “likes”), its total number of reviews, and their average score.
- Instead, given a review, the database must efficiently provide the hotel name, its number of stars and its city.
```python
hotel{
  _id: ObjectId(),
  name: <string>,
  stars: <number>,
  services: [ <string> ],
  venue: {
    address: <string>,
    city: <string>,
    country: <string>,
    website: <url>
  },
  top_reviews : [
    {_id: ObjectId(),
     timestamp: <date>,
     score: <number>,
     nickname: <string>,
     likes: <number>,
     description: <string>
    }, ...
  ],
  tot_reviews: <number>,
  avg_score: <number>
}
# Polymorphic pattern to track the venue information in the hotel collection (due to the optional website info).
# Subset pattern to track the top 10 reviews for each hotel.
# Computed pattern for the average score and total review count of each hotel.
review{
  _id: ObjectId(),
  timestamp: <date>,
  score: <number>,
  nickname: <string>,
  likes: <number>,
  description: <string>,
  hotel: {
    id: ObjectId(),
    name: <string>,
    stars: <number>,
    city: <string>,
  }
}
# Extended reference for the review collection to show the hotel info.
```
## January 28, 2022
Design a MongoDB database to store offers of merchants for a mobile app according to the following requirements.
- Data to display for each merchant includes the merchant's name, type of merchant (e.g., restaurant, hotel, supermarket), a textual description, the venue, the average rating given by customers, and the contact information.
- The venue consists of the full address, the city, ZIP code, and country.
- Contact information includes the phone number and email address. The official website address and the Facebook page might be included in the contact information.
- Each offer consists of a title, a textual description, a list of categories (e.g., wellness, food, wine), a price in euros, and the validity period (i.e., start and end dates).
- Each offer is related to a specific merchant. A merchant can have many offers.
- Given a merchant, the database must be designed to efficiently provide all the data describing the merchant, the number of available offers, and their price range (i.e., min and max prices). Instead, given an offer, the database must efficiently provide the merchant name, the merchant type and its venue information.

Write a sample document for each collection of the database.
```python
merchant{
  _id: ObjectId(),
  name: <string>,
  type: <string>,
  description: <string>,
  venue: {
    address: <string>,
    city: <string>,
    ZIP: <number>
    country: <string>
  },
  rating: <number>,
  contact_info: {
    phone: <string>,
    email: <string>,
    website: <url>
    facebook: <url>
  },
  available_offers: <number>,
  price_range: {min: <number>, max: <number>}
}
# Polymorphic pattern to track the contact information in the merchant collection (due to the optional website and facebook info).
# Computed pattern for the total number of offers and the price range available foreach merchant.
offer{
  _id: ObjectId(),
  title: <string>,
  description: <string>,
  categories: [ <string> ],
  price: <number>,
  start: <date>, 
  end: <date>,
  merchant: {
    _id: ObjectId(),
    name: <string>,
    type: <string>,
    venue: {
      address: <string>,
      city: <string>,
      ZIP: <number>
      country: <string>
    }
  }
}
# Extended reference for the offers collection to show the merchant info.
```
