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
