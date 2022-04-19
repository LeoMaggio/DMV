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
