# Theory Questions
1. During the election of the primary node in a MongoDB replica set, the following operation is allowed:
```db.grades.find({vote:{$gte:8}})```
2. The attribute pattern has the advantage of: requiring fewer indexes
3. According to the CAP theorem, two nodes on opposite sides of a partition are: either consistent or available
4. Select which statement regarding the degenerate dimensions is wrong:
    - degenerate dimensions can be directly integrated into the fact table [X]
    - a degenerate dimension is characterized by a set of attributes structured as a hierarchy
    - a degenerate dimension is a dimension with a single attribute
    - multiple degenerate dimensions can be modeled as a junk dimension
5. The document versioning pattern has the disadvantage of: doubling the number of writes
6. The aggregation window defined at the logical level: is appropriate for sequence data with gaps and sparse data
7. A visualization contains the weight of a person reported as 182 cm. With respect to data quality, this is a problem of: Accuracy
8. If a variable represents heights of people and a data point is "0.002 km", we are observing an issue of: Precision
9. In Datawarehouse analysis, the aggregation window defined at the physical level in extended SQL: is defined by counting the rows
10. In NoSQL design, the extended reference pattern has the advantage of: reducing the join operations
11. Select the right configuration of a MongoDB replica set: 1 primary node, 2 secondary nodes , 2 arbiter nodes
12. Which one of the following examples is NOT related to a Gestalt principle?
    - the bars representing smaller values are shorter [X]
    - the points of a data series are connected
    - the color of the legend is similar to the color of the elements of the graph
    - the direct labeling technique improves the readability of the visualization
    - the points of a group are enclosed by a fine line
13. During the ETL process, the correct order of data warehouse loading is: update dimensions, then tables, finally materialized views and indices
14. The approximation pattern has the advantage of: fewer writes to the database
15. In the aggregation pipeline, which stage operator is used to execute a recursive searchon a collection: `$graphLookup`
16. Which one of the following visualizations is the most appropriate one for representing a measure as a statistical distribution, a dimension with a high cardinality and another dimension with a low cardinality? For example, think about a visualization representing incomes (measure), level of education (dimension with high cardinality), and gender (dimension with low cardinality). Multiple box plots
17. Given a collection of documents, each describing a photo, the statement 
```
db.photos.updateMany( {user: "john", tag: "seaside" }, { $addToSet: {tag: "Riccione"} } )
```
adds the tag “Riccione” to all the photos belonging to the user “john” and having the value “seaside” in the tag list

18. In the MongoDB aggregation pipeline, which stage operator is used to output a new document for each element of an array: `$unwind`
19. In a master-slave distributed database setting, when the replication is asynchronous: data can be lost even if the master has already committed
20. Which one of the following answers is a direct consequence of Steven's law? It is important to avoid comparisons between areas
21. An arbiter node of a MongoDB replica set: does not hold data and can participate in elections
22. The `find()` operator in MongoDB: allows to specify both the documents of interest and the specific fields to be returned from a collection
23. Considering the CAP theorem and its evolutions, when a distributed system provides Availability and Partition tolerance: it can become eventually consistent
24. In a list of email addresses, you find a phone number. In the context of data quality, this is an issue of: Accuracy
25. In a replica set configuration the write acknowledgment is returned when: the primary locally applies the write if w:1
26. The query `db.student.find( { "tests": { $elemMatch: { “score”: { $gt: 18, $lte: 24 } } } } )` returns: all the documents where the “tests” array has at least oneembedded document that contains the field “score” greater than 18 and less than orequal to 24
27. Which of the following is a valid reason to use graphs instead of tables? To reveal relationships among multiple values
