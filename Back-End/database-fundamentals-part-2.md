# Data Maniputlation and Modification
-adding, updating, deleting data from a database
INSERT INTO
UPDATE
DELETE


INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);

This inserts a new row into the table. Columns that are not specified in the statement will be filled with the default value or NULL if no default value is specified.

Example:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER
);

INSERT INTO students (first_name, last_name, grade)
VALUES ('John', 'Doe', 90);
INSERT INTO students (first_name, last_name, grade)
VALUES ('Jane', 'Doe', 95);
INSERT INTO students (first_name, last_name, grade)
VALUES ('Sally', 'Smith', 100);

SELECT * FROM students;


UPDATE table_name 
SET column1 = value, column2 = value, ...
WHERE condition;

This statement updates existing rows in the table. The WHERE clause specifies which rows to update. If the WHERE clause is omitted, all rows in the table will be updated.

Example:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER
);

INSERT INTO students (first_name, last_name, grade)
VALUES ('John', 'Doe', 90);
INSERT INTO students (first_name, last_name, grade)
VALUES ('Jane', 'Doe', 95);
INSERT INTO students (first_name, last_name, grade)
VALUES ('Sally', 'Smith', 100);

UPDATE students
SET grade = 100
WHERE first_name = 'John';

SELECT * FROM students;


DELETE FROM table_name
WHERE condition;

This statement deletes existing rows from the table. The WHERE clause specifies which rows to delete. If the WHERE clause is omitted, all rows in the table will be deleted.

Example:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER
);

INSERT INTO students (first_name, last_name, grade)
VALUES ('John', 'Doe', 90);
INSERT INTO students (first_name, last_name, grade)
VALUES ('Jane', 'Doe', 95);
INSERT INTO students (first_name, last_name, grade)
VALUES ('Sally', 'Smith', 100);

DELETE FROM students
WHERE first_name = 'John';

SELECT * FROM students;


Practice Problem:
CREATE TABLE Users (
    id int,
    first_name varchar(255),
    last_name varchar(255),
    email varchar(255),
    password varchar(255)
);

INSERT INTO Users (id, first_name, last_name, email, password)
VALUES (1, 'John', 'Doe', 'johndoe@gmail.com', 'password');
INSERT INTO Users (id, first_name, last_name, email, password)
VALUES (2, 'Jane', 'Doe', 'janedoe@gmail.com', 'password');
INSERT INTO Users (id, first_name, last_name, email, password)
VALUES (3, 'Sally', 'Smith', 'sallysmith@gmail.com', 'password');

UPDATE Users
SET email = 'updatedemail@gmail.com'
WHERE id = 1;

DELETE FROM Users
WHERE id = 2;

SELECT * FROM Users;



# Database Design and Normalization
- database design: designing a database schema, which is a collection of tables that are realted to each other
- normalization: organizing data in a database; the goal is to reduce data redundancy and improve data integrity, ensuring data in the database is accurate and consistent

Database schema design
A relational database:
-each table represents an entity and each row in the table represents an instance of that entity
-each entity has attributes that describe the entity
-the attributes are represented by columns in the table
-each attribute has a data type and a default value
-the data type specifies what kind of data can be stored in the attribute
-the default value specifies what value should be used when no other value is specified

A database schema designs the structure of the database - the tables, columns, data types, relationships between tables. It also defines the constraints applied to the data in the database.

Cardinality
The number of instances of one entity that can be related to one instance of another entity.
One-to-one (1:1)

One-to-many (1:N)

Many-to-many (M:N)

Many-to-one (M:1)

[Look up examples of crow's feet graphical representation for modeling a database structure]


Practice Problem:
Relationships between schools and students
-entities: schools, students
-attributes: school - school_id, name, address, level, number_of_students
        students - id, first_name, last_name, age, grade, school_id
-relationships between entities: which students attend which school (school.school_id, student.school_id)
-cardinalities of relationships: school-students one-to-many; students-to-school many-to-one



Primary and foreign keys
A foreign key is a column that references a column in another table.
It's used to establish a relationship between the two tables, to ensure the data is valid, and to enfore referential intergrity.

Example of one-to-many (one school can have many students):
CREATE TABLE schools (
  id INTEGER PRIMARY KEY,
  name TEXT,
  address TEXT,
  phone_number TEXT,
  website TEXT,
  number_of_students INTEGER
);

CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER,
  school_id INTEGER,
  FOREIGN KEY (school_id) REFERENCES schools(id)
);

INSERT INTO schools (name, address, phone_number, website, number_of_students)
VALUES ('School 1', '123 Main St', '123-456-7890', 'school1.com', 100);
INSERT INTO schools (name, address, phone_number, website, number_of_students)
VALUES ('School 2', '456 Main St', '123-456-7890', 'school2.com', 200);

INSERT INTO students (first_name, last_name, grade, school_id)
VALUES ('John', 'Doe', 90, 1);
INSERT INTO students (first_name, last_name, grade, school_id)
VALUES ('Jane', 'Doe', 95, 1);
INSERT INTO students (first_name, last_name, grade, school_id)
VALUES ('Sally', 'Smith', 100, 2);

SELECT * FROM students
WHERE school_id = 1;


Example of many-to-many (one product can be in many orders and one order can have many products):
CREATE TABLE products (
  id INTEGER PRIMARY KEY,
  name TEXT,
  price REAL
);

CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  customer_id INTEGER,
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE order_items (
  id INTEGER PRIMARY KEY,
  order_id INTEGER,
  product_id INTEGER,
  FOREIGN KEY (order_id) REFERENCES orders(id),
  FOREIGN KEY (product_id) REFERENCES products(id)
);

INSERT INTO products (name, price)
VALUES ('Product 1', 10.00);
INSERT INTO products (name, price)
VALUES ('Product 2', 20.00);
INSERT INTO products (name, price)
VALUES ('Product 3', 30.00);

INSERT INTO orders (customer_id)
VALUES (1);
INSERT INTO orders (customer_id)
VALUES (2);

INSERT INTO order_items (order_id, product_id)
VALUES (1, 1);
INSERT INTO order_items (order_id, product_id)
VALUES (1, 2);

INSERT INTO order_items (order_id, product_id)
VALUES (2, 2);
INSERT INTO order_items (order_id, product_id)
VALUES (2, 3);

SELECT * FROM products
WHERE id IN (
  SELECT product_id FROM order_items
  WHERE order_id = 1
);
This uses a subquery to get all the products that are in order 1.


Normalization
Organizing data in a database; the goal is to reduce data redundancy and improve data intergrity.
Different levels of normalization (first 3 are the most common):

First normal form (1NF) - 
Most basic level; used to ensure data in the database is atomic, meaning each attribute in a table contains only one value. And each row in a table contains only one instance of an entity.

Example of non-atomic table:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER,
  address TEXT
);

The address attribute contains multiple values, representing the street address, city, state, zip code. This will make it very difficult to query the databse.

To make the table atomic:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER
);

CREATE TABLE addresses (
  id INTEGER PRIMARY KEY,
  student_id INTEGER,
  street TEXT,
  city TEXT,
  state TEXT,
  zip_code TEXT,
  FOREIGN KEY (student_id) REFERENCES students(id)
);

Second normal form (2NF) - 
Used to ensure data in the database is in the right place; and to ensure that each attribute in a table is dependent on the primary key.

Example of a table with data not in the right place:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER,
  school_name TEXT,
  school_address TEXT
);

The school_name and school_address are not dependent on the primary key. If we wanted to update the school_name or school_address, we'd have to update each row in the database. We'd also have to ensure that school_name and school_address are correct for each row in the table; there could be mistakes.

To make that table in the right place:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER,
  school_id INTEGER,
  FOREIGN KEY (school_id) REFERENCES schools(id)
);

CREATE TABLE schools (
  id INTEGER PRIMARY KEY,
  name TEXT,
  address TEXT
);


Third normal form (3NF) - 
Used to ensure data in the database is not redundant; and to ensure each attribute in a table is dependent on the primary key.

An example that's redundant:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER,
  school_name TEXT,
  school_address TEXT
  school_phone_number TEXT
  school_website TEXT
);

To make it not redundant:
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  grade INTEGER,
  school_id INTEGER,
  FOREIGN KEY (school_id) REFERENCES schools(id)
);

CREATE TABLE schools (
  id INTEGER PRIMARY KEY,
  name TEXT,
  address TEXT,
  phone_number TEXT,
  website TEXT
);

Boyce-Codd normal form (BCNF)
Fourth normal form (4NF)
Fifth noraml form (5NF)
Domain/key normal form (DKNF)


Practice problem:
Normalize the following table.
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  email TEXT,
  password TEXT,
  addresses TEXT,
  phone_numbers TEXT
);

My solution:
CREATE TABLE users (
    id INTERGER PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    email TEXT,
    password TEXT
)

CREATE TABLE addresses (
  id INTEGER PRIMARY KEY,
  user_id INTEGER,
  street TEXT,
  city TEXT,
  state TEXT,
  zip_code TEXT,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE phone_numbers (
  id INTEGER PRIMARY KEY,
  user_id INTEGER,
  phone_number TEXT,
  FOREIGN KEY (user_id) REFERENCES users(id)
);



# Security and Best Practices
Database security: protecting the database from unauthorized access, misuse, or data breaches.
Performance optimization: making the database perform operations more efficiently.
Database design and normalization: structuring a database to minimize redundancy and dependency.
Database schema design: the layout of the database that defines how data is organized into tables and relationships.
Database administration: the role of managing and maintaining database systems.
Database backup and recovery: processes to safeguard data by creating copies and restoring dat from those backups when necessary.


# Installing Ruby




# Video - Crows Foot ERD & Cardinality
ERD: entity relationship diagram
Help us plan out the structure of our database.

https://www.lucidchart.com/pages/

Click on Templates, then choose Database ER diagram
Click and drag Entity Relationship filed boxes.

Entity for User:
primary key user_id number
            email string
            password string

Primary key - unique id to select a specific entity

Entity for Book:
primary key book_id number
            title string
            author string
            publish_year date
            genre string
            cover_image string
foreign key user_id number

Because users can save books, a book has a foreign key that relates the book to a user.
A user can have 0 to many books.
A book can have 0 to one user.


Entity for Genre:
primary key id number
            slug string
foreign key book_id number

A slug is a placeholder, so we don't have to have an array of a bunch of genres.
Each genre will be a whole row that has a slug, and the slug is a string that says something like "mystery" or "science fiction".
The slug will be whatever data is input for that book.
It's a way to create a scaleable genre category for an app.
[look up more about this]

Here we'll say a book can be attached to one or many genres, but each genre is attached to only one book.

User table:
user_id | email | password
30001     test@test.com     #8flj*6

Book table:
book_id | title | author | first_published | user_id
1234    Harry Potter    JK Rowling  1999    30001

[the user_id would probably be an array because multiple users could have a book saved on their bookshelf - we'll go over that in a later lesson]

Genre table:
id | slug | book_id
123 mystery 1234

[the book_id would also be an array of multiple books, and the genre would be like a tag on the book; when a user searches for mystery it would find all books with the mystery tag attached to it]

Setting it up with a slug makes it easier to create new rows later on.

In the ERD the entities are set up with attributes in a column and those rows in the column become the column headers for the tables in the database. Then each entity is a row with those attributes. 


# Notes from class 1/25/24:
To create diagrams (ERDs):
app.diagrams.net
Lucid charts
plugin for VS Code
https://marketplace.visualstudio.com/items?itemName=dineug.vuerd-vscode

(1:1...N): an author can have one or more books and a book belongs to an author
[the author needs to have at least one book to be considered an author]
(1:0...N): a publisher can have zero to many books and a books belongs to a publisher

In the chart, we use the one-to-many line and a mandatory symbol - it's mandatory for a book to be associated with an author.

An author can have many publishers (through books) and a publisher can have many authors (through books).

CREATE TABLE Authors (
  AuthorID INT PRIMARY KEY,
  name TEXT,
  Bio TEXT
);

CREATE TABLE Publishers (
  PublisherID INT PRIMARY KEY,
  Name TEXT,
  Address TEXT
);

CREATE TABLE Books (
  BookID INT PRIMARY KEY,
  Title TEXT,
  AuthorID INT,
  PublisherID INT,
  FOREIGN KEY (AuthorID) REFERENCES AUTHORS(AuthorID),
  FOREIGN KEY (PublisherID) REFERENCES PUBLISHERS(PublisherID)
);

INSERT INTO Books(BookID, Title, AuthorID, PublisherID)
VALUES(
  1,
  'Carry On',
  1,
  1
);
[also INSERT INTO Authors and Publishers]

SELECT
  Authors.Name AS AuthorName,
  Publishers.Name AS PublisherName,
  Books.Title
FROM
  Books
INNER JOIN Authors ON Books.AuthorID = Authors.AuthorID
INNER JOIN Publishers On Books.PublisherID = Publishers.PublisherID;

Output: Rainbow Rowell | Noon Virtual Group | Carry On