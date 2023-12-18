# SQL-Structured Query Language
## Data:
- Collection of facts and information
- structured like tabular format, excel
- unstructured like image, audio, video
- semi-structured like JSON, XML

## Database:
- It is a place where data is stored in an organized manner

## DBMS:
- It is a tool/software that allows one to access, interact with, and manipulate a database
- RDBMS use SQL to communicate with database

## Types of SQL Commands
**Data Query Language (DQL) (retrieve data from the DB using SQL queries):**
- SELECT

**Data Definition Language (DDL) (define database schema in DBMS):**
- CREATE
- ALTER
- DROP
- TRUNCATE
  
**Data Manipulation Language (DML) (manipulate data present in the DB):**
- INSERT
- UPDATE
- DELETE
- MERGE
  
**Data Control Language (DCL)  (deals with access rights and data control on the data present in the db):**
- GRANT
- REVOKE
  
**Transaction Control Language (TCL)  (deals with the transactions happening in the DB):**
- COMMIT
- ROLLBACK
- SAVEPOINT

## Difference between Delete vs Drop vs Truncate?
- DELETE is used to remove specific rows based on a condition and is a DML command.
- DROP is used to remove an entire database object, such as a table, and is a DDL command.
- TRUNCATE is used to remove all rows from a table and is also a DDL command, but it is faster than DELETE for removing all rows from a table.

## Types of Keys in SQL
**Primary Key:**
- Unique identifier for a record in a table.
- Ensures data integrity and enforces entity integrity.
- Cannot contain NULL values.

**Foreign Key:**
- Column(s) that refer to the primary key in another table.
- Establishes a link between tables, creating a relationship.
- Ensures referential integrity.

**Unique Key:**
- Ensures that all values in a column or a set of columns are unique.
- Allows NULL values except in cases where the column is also a primary key.

**Candidate Key:**
- A column or a set of columns that can qualify as a primary key.
- Meets the criteria for being a key but is not chosen as the primary key.

**Super Key:**
- A set of one or more columns that, taken collectively, can uniquely identify a record in a table.
- May include more columns than necessary to uniquely identify a record.

**Composite Key:**
- A primary key that consists of multiple columns.
- Combines the values of those columns to create a unique identifier.

**Alternate Key:**
- A candidate key that is not selected as the primary key.
- Provides an alternative unique identifier for a record.

## Relations in Database
**One-to-One Relationship:**
- Each record in the first table is related to only one record in the second table, and vice versa.
- Example: Person and Passport, where each person has one passport, and each passport is associated with one person.

**One-to-Many Relationship:**
- A record in the first table can be related to one or more records in the second table, but a record in the second table is related to only one record in the first table.
- Example: Customer and Orders, where one customer can place multiple orders, but each order is associated with only one customer.

**Many-to-One Relationship:**
- Opposite of a one-to-many relationship; many records in the first table can be related to one record in the second table.
- Example: Many employees (from different departments) report to one manager in a company.

**Many-to-Many Relationship:**
- Many records in the first table can be related to many records in the second table.
- Requires a junction table to implement the relationship.
- Example: Students and Courses, where each student can enroll in multiple courses, and each course can have multiple students.

## Order of execution
- FROM, including JOINs
- WHERE
- GROUP BY
- AGGREGATE functions
- HAVING
- WINDOW functions
- SELECT
- DISTINCT
- UNION
- ORDER BY
- LIMIT

## CASE Statement
```SQL:
  SELECT 
    market_day,  
    CASE  
      WHEN market_day IN ('Saturday' , 'Sunday’) THEN 1    
      ELSE 0    
    END AS weekend_flag  
  FROM 
    farmers_market.market_date_info

  SELECT
    market_date,  
    customer_id,  
    vendor_id,  
    ROUND(quantity * cost_to_customer_per_qty, 2) AS price,  
    CASE  
      WHEN quantity * cost_to_customer_per_qty < 5.00 THEN 'Under $5'
      WHEN quantity * cost_to_customer_per_qty < 10.00 THEN '$5-$9.99'
      WHEN quantity * cost_to_customer_per_qty < 20.00 THEN '$10-$19.99'
      WHEN quantity * cost_to_customer_per_qty >= 20.00 THEN '$20 and Up'    
    END AS price_bin  
  FROM 
    farmers_market.customer_purchases
```
## Rules for Group by
1. column list in GROUP by and SELECT should match
2. Any additional column in select clause must be within the aggregate functions
3. select clause only has aggregate functions
4. group by columns should either be present in select clause as standalone columns or within the aggregate functions
```SQL:
SELECT product_line, count(1)
FROM products
GROUP BY product_code;
-- product_code is primary key so this will work as per point 4
```

## Window Functions
Window fns give the ability to put the values from one row of data into context compared to a group of rows, or partition.

We can answer questions like
- If the dataset were sorted, where would this row land in the results?
- How does a value in this row compare to a value in the prior row?
- How does a value in the current row compare to the average value for its group?

So, window functions **return group aggregate calculations alongside individual row-level information** for items in that group, or partition. 

**Order of execution for window function:**
1. Partition by
2. Order by
3. window funciton itself

## Views vs CTEs

**Common Table Expression (CTE):**
- A CTE is a temporary result set that is defined within the scope of a single SQL query.
- It is typically used to simplify complex queries, break them down into smaller, more manageable parts, and make the code more readable.
- CTEs are defined using the WITH keyword, followed by the CTE name and the query that defines the result set.
- CTEs are only visible to the query in which they are defined. They cannot be referenced by other queries or stored in the database like views.
- CTEs are usually more performant than views for certain scenarios since they are materialized and optimized within the scope of the main query.

**Views:**
- A view is a virtual table created by saving the result set of a SELECT query in the database. 
- It is a named object that stores the query definition and can be treated like a regular table in subsequent queries.
- Views are used to simplify data access by providing an additional level of abstraction. They hide the underlying complexity of the database schema and can present a specific subset of data to users without revealing the actual table structure.
- Views are stored in the database and can be referenced by multiple queries from different parts of the application.
- Views can be more efficient for certain scenarios where the query results are frequently reused since they are precomputed and stored in the database.

**Types of Views:**
- **Simple View:** A simple view is based on a single table and contains a SELECT statement. It may include only specific columns from the table and can have filtering conditions (WHERE clause).
- **Complex View:** A complex view is based on multiple tables or other views. It can include JOINs and aggregate functions, providing a consolidated and structured view of data from different tables.
- **Indexed View (Materialized View):** Some database systems support indexed views, which are views that have an associated index. The index improves the performance of queries that directly match the view's SELECT statement.
- **Partitioned View:** A partitioned view is used to partition a large table horizontally into smaller, manageable pieces based on some criteria.

Although there are some differences between them, common table expressions and views seem to perform very similarly. So, when should you use each one?

- **Ad-hoc queries:** For queries that are referenced occasionally (or just once), it’s usually better to use a CTE. If you need the query again, you can just copy the CTE and modify it if necessary.
- **Frequently used queries:** If you tend to reference the same query often, creating a corresponding view is a good idea. However, you’ll need create view permission in your database to create a view.
- **Access management:** A view might be used to restrict particular users’ database access while still allowing them to get the information they need. You can give users access to specific views that query the data they’re allowed to see without exposing the whole database. In such a case, a view provides an additional access layer.

## Unions

- Using a UNION, you can combine any two queries that result in the same number of columns with the same data types.
- The columns must be in the same order in both queries.
- There are many possible use cases for UNION queries, but the syntax is simple: write two queries with the same number and type of fields, and put a UNION keyword between them:
- Let’s say you want to get all the cities from two different tables.
```SQL:
  SELECT City FROM Customers
  UNION
  SELECT City FROM Suppliers
  ORDER BY City;
```
**To retain the duplicate rows:**
```SQL:
  SELECT City FROM Customers
  UNION ALL
  SELECT City FROM Suppliers
  ORDER BY City;
```
**To get common records without duplicates:**
```SQL:
  SELECT City FROM Customers
  INTERSECT
  SELECT City FROM Suppliers
  ORDER BY City;
```
**To get unique records only from first Table:**
```SQL:
  SELECT City FROM Customers
  EXCEPT
  SELECT City FROM Suppliers
  ORDER BY City;
```

## SQL types of Joins
- JOIN combines data from two tables.
- JOIN typically combines rows with equal values for the specified columns. Usually, one table contains a primary key, which is a column or columns that uniquely identify rows in the table. The other table has a column or columns that refer to the primary key columns in the first table. Such columns are foreign keys.
- The JOIN condition doesn't have to be an equality – it can be any condition you want. JOIN doesn't interpret the JOIN condition, it only checks if the rows satisfy the given condition.
- You can join more than two tables together. First, two tables are joined, then the third table is joined to the result of the previous joining.
- You can use multiple JOIN conditions using the ON keyword once and the AND keywords as many times as you need.

**INNER JOIN:**
- JOIN (or explicitly INNER JOIN) returns rows that have matching values in both tables.

```SQL:
SELECT *
FROM table_1 t1
INNER JOIN table_2 t2 ON t1.id = t2.id;
```

![sample_image]()

**LEFT JOIN:**
- LEFT JOIN returns all rows from the left table with matching rows from the right table. Rows without a match are filled with NULLs. LEFT JOIN is also called LEFT OUTER JOIN.

```SQL:
SELECT *
FROM table_1 t1
LEFT JOIN table_2 t2 ON t1.id = t2.id;
```

![sample_image]()

**RIGHT JOIN:**
- RIGHT JOIN returns all rows from the right table with matching rows from the left table. Rows without a match are filled with NULLs. RIGHT JOIN is also called RIGHT OUTER JOIN.

```SQL:
SELECT *
FROM table_1 t1
RIGHT JOIN table_2 t2 ON t1.id = t2.id;
```

![sample_image]()

**FULL JOIN:**
- FULL JOIN returns all rows from the left table and all rows from the right table. It fills the non-matching rows with NULLs. FULL JOIN is also called FULL OUTER JOIN.

```SQL:
SELECT *
FROM table_1 t1
FULL JOIN table_2 t2 ON t1.id = t2.id;
```

![sample_image]()

**CROSS JOIN:**
- CROSS JOIN returns all possible combinations of rows from the left and right tables.
```SQL:
SELECT *
FROM table_1 t1
CROSS JOIN table_2 t2;

-- non ANSI way
SELECT *
FROM table_1 t1, table_2 t2;
```

id	id-2
1	1
1	1
1	2
1	2
1	4
1	NULL
1	1
1	1
1	2
1	2
1	4
1	NULL
1	1
1	1
1	2
1	2
1	4
1	NULL
2	1
2	1
2	2
2	2
2	4
2	NULL
3	1
3	1
3	2
3	2
3	4
3	NULL
3	1
3	1
3	2
3	2
3	4
3	NULL
3	1
3	1
3	2
3	2
3	4
3	NULL

**NATURAL JOIN:**
- NATURAL JOIN will join tables by all columns with the same name and same data type.
- If some other columns with same name exist then it will mess with logic of joining tables.
```SQL:
SELECT *
FROM table_1 t1
NATURAL JOIN table_2 t2;
```

![sample_image]()

## LIKE OPERATOR – PATTERN MATCHING
- Use the _ character to identify any single character.
- Use the % character to identify any number of characters (including 0 characters).

## CASTING
- It lets you change the type of value to almost anything (integer, numeric, double precision, varchar, and many more).
- SELECT CAST(column_name AS DATA_TYPE);

## SUBQUERIES
- A subquery is a query that is nested inside another query, or inside another subquery

**SINGLE VALUE SUBQUERY**
- The simplest subquery returns exactly one column and exactly one row. It can be used with comparison operators =, <, <=, >, or >=.

**MULTIPLE VALUES SUBQUERY**
- A subquery can also return multiple columns or multiple rows. Such subqueries can be used with operators IN, EXISTS, ALL, or ANY.

**CORRELATED SUBQUERY**
- A correlated subquery refers to the tables introduced in the outer query. A correlated subquery depends on the outer query. It cannot be run independently from the outer query.
```SQL:
  SELECT *
  FROM city main_city
  WHERE population > (
     SELECT AVG(population)
     FROM city average_city
     WHERE average_city.country_id = main_city.country_id
    );
```
## What is Indexing and different type of indexes?
- In SQL, indexing is a technique used to improve the performance of queries by creating data structures that allow for faster data retrieval.
- Indexes are created on specific columns of a table, and they help the database engine locate the desired data more efficiently, reducing the need for full-table scans or searching through all records.
- **Single-Column Index:** A single-column index is created on a single column of a table. It speeds up queries that involve conditions or sorting based on that particular column.
```SQL:
CREATE INDEX index_name ON table_name (column_name);
```
- **Composite Index (Multi-Column Index):** A composite index is created on multiple columns of a table. It is useful when queries involve conditions on multiple columns.
```SQL:
CREATE INDEX index_name ON table_name (column1, column2, ...);
```
- **Unique Index:** A unique index enforces the uniqueness of values in one or more columns of a table. It is often used to enforce the primary key or unique constraint.
```SQL:
CREATE UNIQUE INDEX index_name ON table_name (column_name);
```
- **Clustered Index:** In SQL Server, a clustered index determines the physical order of data in a table. Each table can have only one clustered index. In other database systems like MySQL, the primary key is used as the clustered index by default.
```SQL:
CREATE CLUSTERED INDEX index_name ON table_name (column_name);
```
- **Non-Clustered Index:** A non-clustered index is a separate data structure from the table that contains a sorted copy of the indexed column(s) and a reference to the actual row in the table. A table can have multiple non-clustered indexes.
```SQL:
CREATE INDEX index_name ON table_name (column_name);
```
