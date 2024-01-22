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




# Database Design and Normalization

# Security and Best Practices

# Installing Ruby