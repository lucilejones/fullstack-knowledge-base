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

# Introduction to SQL
Structured Query Language
Used for retrieving data, inserting new data, updating existing data, deleting data.
Similar to English - relatively easy to write, read, interpret
Most SQL syntax is compatible with all SQL databases.


Creating tables:
A table is a collection of related data entries; consists of columns and rows
Online SQL editor: https://www.jdoodle.com/execute-sql-online

We use the CREATE TABLE statement:
CREATE TABLE Employees (
    EmployeeID int,
    LastName varchar(255),
    FirstName varchar(255),
    BirthDate date,
    Notes text
);

int - used for interger numbers
varchar - used for variable-length character strings
date - used for date values
text - used for long text strings

Other data types:
float - used for floating-point numbers
decimal - used for decimal numbers
boolean - used for boolean values (true/false)
blob - used for binary data
json - used for storing JSON data
uuid - used for storing UUIDs
array - used for storing arrays

Size of the data definitely matters because it affects the performance of the database.
If we know a column will only store values up to 255 characters, we should use varchar(255) instead of text. If we know a column will only store values up to 10 characters, we should use varchar(10) instead of varchar(255).
The database will allocate less memory for that column.

In the create table above, the columns are:
EmployeeID - an integer number that uniquely identifies each employee
LastName - a variable-length character string that stores the last name of the employee
FirstName - a variable-length character string that stores the first name of the employee
BirthDate - a date value that stores the birth day of the employee
Notes - a long text string that stores notes about the employee


Inserting data:

We use the INSERT INTO statement:
INSERT INTO Employees (EmployeeID, LastName, FirstName, BirthDate, Notes)
VALUES (1, 'Doe', 'John', '1990-01-01', 'John Doe was born on January 1, 1990.');
INSERT INTO Employees (EmployeeID, LastName, FirstName, BirthDate, Notes)
VALUES (2, 'Doe', 'Jane', '1991-02-02', 'Jane Doe was born on February 2, 1991.');

INSERT INTO - adds the specified row or rows into the table
Employees - the name of the table
(EmployeeID, LastName, FirstName, BirthDate, Notes) - a parameter that lists the columns that the INSERT INTO statement will insert values into
VALUES - indicates the data being inserted in the order specified in the parameter list


Selecting Data:
Retrieving data from a database; also called a query

We use the SELECT statement:
SELECT * FROM Employees;

This will give the following output:
1|Doe|John|1990-01-01|John Doe was born on January 1, 1990.

SELECT - indicates the statement is a query; we'll use SELECT every time we query data from a database
* - a wildcard character that indicates all columns
FROM Employees - indicates the table we're querying data from
; - indicates the end of the statement (optional, but good practice)

A query always returns a result set, even if it's empty. We can think of a result set as a temporary table that holds the data returned by the query. In the example above, the result set contains all rows from the Empoyees table.

If we want to select only some columns we can specify them after SELECT. 
Example, selecting only FirstName and LastName from the Employees table:
SELECT FirstName, LastName FROM Employees;

This will output:
John|Doe


Where Clause:
Used to filter records. We can extract only those records that fulfill a specified condition.
The following selects all rows from the Employees tables where FirstName is equal to John:
SELECT * FROM Employees WHERE FirstName = 'John';

WHERE - indicated we want to filter the result set to inclue only rows where the following condition is true
FirstName - use this to set the condition

The WHERE clause can be combined with AND, OR, and NOT operators.

SELECT * FROM Employees WHERE FirstName = 'John' AND LastName = 'Doe';

AND - combines two conditions; both must be true for the row to be included in the result set

SELECT * FROM Employees WHERE FirstName = 'John' OR LastName = 'Doe';

OR - combines two conditions; either one must be true for the row to be included

SELECT * FROM Employees WHERE NOT FirstName = 'John';

NOT - this operator negates the condition that follows it; displays a record if the condtion is not true

The WHERE clause allows us to extract only the data we need from a table. This reduces the load on the database server. We can limit the query to data that meets a specific condtion, or we can limit the number of rows returned.
We can also limit the data a user can access. For example, only seeing their own data. This is called row-level security.


Order By Clause:
Used to sort the result set in ascending or descending order. (ORDER BY sorts in ascending order by default. To sort in descending order, we use the DESC keyword).

SELECT * FROM Employees ORDER BY FirstName;

Gives the following output:
2|Doe|Jane|1991-02-02|Jane Doe was born on February 2, 1991.
1|Doe|John|1990-01-01|John Doe was born on January 1, 1990.

ORDER BY - indicates we want to sort the result set
FirstName - the column we want to sort by
SELECT * FROM Employees - the statement we want to execute

SELECT * FROM Employees ORDER BY FirstName DESC; # will output in descending order instead


Limit Clause:
Used to limit the number of rows returned by a query; used in the SELECT statement.

SELECT * FROM Employees LIMIT 3;

To get the last 3 rows we use the OFFSET clause:
SELECT * FROM Employees ORDER BY EmployeeID DESC LIMIT 3;

LIMIT - lets us specify the maximum number of rows the result set will have
3 - the maximum number of rows the result set will have
SELECT * FROM Employees - the statement we want to execute
ORDER BY EmployeeID DESC - sorts the result set in descending order by the EmployeeID column
It's executed from left to right: it will first order the result set in descending order by the EmployeeID column and then it will select the first 3 rows from the result set.


Example table (practice):
Select all rows from the Students table where the Major is equal to Computer Science and the GPA is greater than or equal to 3.5. Sort the result set in descending order by the GPA column. Limit the result set to 1 row.

CREATE TABLE Students (
    StudentID int,
    LastName varchar(255),
    FirstName varchar(255),
    BirthDate date,
    Notes text,
    GPA float,
    Major varchar(50)
);

INSERT INTO Students (StudentID, LastName, FirstName, BirthDate, Notes, GPA, Major)
VALUES (1, 'Doe', 'John', '1990-01-01', 'John Doe was born on January 1, 1990.', 3.5, 'Computer Science');
INSERT INTO Students (StudentID, LastName, FirstName, BirthDate, Notes, GPA, Major)
VALUES (2, 'Doe', 'Jane', '1991-02-02', 'Jane Doe was born on February 2, 1991.', 3.6, 'Computer Science');

SELECT * FROM Students WHERE Major = 'Computer Science' AND GPA >= 3.5 ORDER BY GPA DESC LIMIT 1


Aggregate Functions:
Used to perform calculations on a set of values and return a single value.
Often used with the GROUP BY clause of the SELECT statement.

Most commonly used aggregate functions:
AVG() - returns the average value
COUNT() - returns the number of rows
FIRST() - returns the first value
LAST() - returns the last value
MAX() - returns the largest value
MIN() - returns the smallest value
SUM() - returns the sum

SELECT COUNT(*) FROM Employees;
This returns the number of rows in the Employees table (the output is 2).

SELECT AVG(BirthDate) FROM Employees;
This returns the average birth date of all employees in the Employees table (output 1990.2)

SELECT MAX(BirthDate) FROM Employees;
This returns the largest birth day of employees in the Employees table (output 1991-02-02)

Aggregate functions allow us to perform calculations on a set of values and return a single value.


GROUP BY Clause:
Used to group the result set by one or more columns. 
Often used with aggregate functions like AVG(), COUNT(), MAX(), MIN(), and SUM().

SELECT FirstName, COUNT(*) FROM Employees GROUP BY FirstName;
This groups the result set by the FirstName column and returns the number of employees for each group.

SELECT FirstName, COUNT(*) FROM Employees GROUP BY FirstName ORDER BY COUNT(*) DESC;
This groups the result set by the FirstName column and returns the number of employees for each group, and also sorts the groups in descending order by number of employees.


HAVING clause:
Used to filter groups in the result set.
Often used with aggregate functions like AVG(), COUNT(), MAX(), MIN(), and SUM()

SELECT FirstName, COUNT(*) FROM Employees GROUP BY FirstName HAVING COUNT(*) > 1;
This groups the result set by the FirstName column and returns the number of employees for each group; and it filters the groups that have more than 1 employee.


Example (practice):
Select all rows from the Students table and return the average GPA of all students. Group the result set by the FirstName column and return the average GPA for each group. Filter the groups that have more than 1 student. Sort the groups in descending order by the average GPA.

CREATE TABLE Students (
    StudentID int,
    LastName varchar(255),
    FirstName varchar(255),
    BirthDate date,
    Notes text,
    GPA float
);

INSERT INTO Students (StudentID, LastName, FirstName, BirthDate, Notes, GPA)
VALUES (1, 'Doe', 'John', '1990-01-01', 'John Doe was born on January 1, 1990.', 3.5);
INSERT INTO Students (StudentID, LastName, FirstName, BirthDate, Notes, GPA)
VALUES (2, 'Doe', 'Jane', '1991-02-02', 'Jane Doe was born on February 2, 1991.', 3.6);
INSERT INTO Students (StudentID, LastName, FirstName, BirthDate, Notes, GPA)
VALUES (3, 'Moe', 'John', '1992-03-03', 'Mary Doe was born on March 3, 1992.', 3.7);
INSERT INTO Students (StudentID, LastName, FirstName, BirthDate, Notes, GPA)
VALUES (4, 'Doe', 'Mark', '1993-04-04', 'Mark Doe was born on April 4, 1993.', 3.8);
INSERT INTO Students (StudentID, LastName, FirstName, BirthDate, Notes, GPA)
VALUES (5, 'Doe', 'Lisa', '1994-05-05', 'Lisa Does was born on May 5, 1884.', 3.9);

SELECT FirstName, AVG(GPA) FROM Students GROUP BY FirstName HAVING COUNT(*) > 1 ORDER BY AVG(GPA) DESC;


Introduction to Joins:
A join clause combines rows from two of more tables based on a related table between them.

For example, we have an Employee table with the following columns:
EmployeeID, LastName, FirstName, DepartmentID, BirthDate, Notes, Salaray
And a Departments table with the following columns:
DepartmentID, DepartmentName, ManagerID, Location, Budget, Notes, Employees

The Employees table has a column DepartmentID that is related to the DepartmentID column in the Departments table. This is called a foreign key. 
A foreign key is a column or a group of columns in a table that uniquely identifies a row in another table. This creates a relationship between the two tables. 
A one-to-many relationship: one row can be related to many rows in another table.
For examples, one department can have many employees.

If we use the following statement:
SELECT * FROM Employees, Departments
this will perform a cross join (also known as a Cartesian join); each row from the first table is combined with each row from the second table, resulting in every possible combination of rows between the two tables. 
With 2 rows in Employees and 2 rows in Departments, the cross join will produce 2 * 2 = 4.

However, this isn't super useful. There are other joins that will be more helpful.


Inner Join:
An inner join returns rows when there is a match in both tables. 
The following statement joins the Employees table with the Departments table based on the DepartmentID column:
SELECT * FROM Employees INNER JOIN Departments ON Employees.DepartmentID = Departments.DepartmentID;

ON Employees.DepartmentID = Departments.DepartmentID specifies the columns that we want to join on.
An inner join returns rows when there is a match in both tables. The above statement returns both employees that are in the Sales department including the department info. 

1|Doe|John|1|1990-01-01|John Doe was born on January 1, 1990.|50000|1|Sales|1|New York|1000000|The Sales department is located in New York.|10
2|Doe|Jane|1|1991-02-02|Jane Doe was born on February 2, 1991.|60000|1|Sales|1|New York|1000000|The Sales department is located in New York.|10


Left Join:
A left join returns all rows from the left table and the matched rows from the right table. It's different from an inner join, because it will return all rows from the left table even if there isn't a match in the right table.
The left table is the table specified before the LEFT JOIN clause. 
The right table is the table specified after the LEFT JOIN clause.

[look for a better example than the one in the notes]


Right Join:
A right join returns all rows from the right table and the matched rows from the left table. It's different from an inner join because it will return all rows from the right table even if there's not match in the left table.


Full Join:
A full join returns all rows when there is a match in one of the tables. If there is a match between the two tables, it combines columns from the two tables into a single row.


Self Join:
A self join is used to join a table to itself.
SELECT * FROM Employees e1 INNER JOIN Employees e2 ON e1.DepartmentID = e2.DepartmentID;

e1 - an alias for the Employees table
e2 - an alias for the Employees table


# Bookstore practice problem

CREATE TABLE Authors (
    AuthorID int,
    AuthorName varchar(255)
)

CREATE TABLE Books (
    BookID int,
    Title varchar(255),
    AuthorID int,
    PublicationYear int
)

CREATE TABLE Sales (
    SaleID int,
    BookID int,
    QuantitySold int,
    SaleDate date
);