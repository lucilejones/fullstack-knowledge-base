# Introduction to Databases
- a collection of data stored in a computer system
- store data in a structured way, so it can be easily managed and accessed

SQL - structured query language

# Database Management Systems
DBMS - a software system that uses a standard method to store and organize data.
Data can be added, updated, deleted, or traversed using various standard algorithms and queries.
DBMSs are categorized according to their data structures or types.
The DBMS accepts requests for data from the application program and instructs the operating system to transfer the appropriate data.

# Types of Databases and when to use
There are many different types of databases; two of the most common are relational and non-relational.
Relational are also called SQL databases
-store data in tables
-used for storing structured data; use SQL for defining and manipulating the data
-easy to extend; a new category can be added after the original database creation without requiring a modification to all existing applications
-the most commonly used database type
-examples: MySQL, Oracle
-Project examples: WordPress, Drupal

Non-relational are also called NoSQL databases
-store data in documents, key-value pairs, graph databases, or wide-coumn stores
-used for storing unstructured data
-use a variety of data models, including document, graph, key-value, and columnar
-useful for very large databases and when the data structure isn't fixed (and might change over time)
-do not use SQL
-stored in a format optimized for the specific data being stored
-examples: MongoDB, Cassandra
-Project examples: Netflix, LinkedIn

It's also possible to use both types in the same application: a relational database to store user data and a non-relational database to store user comments.

# Different types of SQL databases
MySQL
the most popular open-source database
generally known for its fast read operations

PostgreSQL
also open-source
generally known for its reliability, data integrity, and correctness
excels in write-heavy operations and complex query optimizations

SQLite
a lightweight database
generally known for its simplicity