# SQL-Structured Query Language

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

## Types of SQL Commands
**Data Query Language (DQL):**
- SELECT

**Data Definition Language (DDL):**
- CREATE
- ALTER
- DROP
- TRUNCATE
  
**Data Manipulation Language (DML):**
- INSERT
- UPDATE
- DELETE
  
**Data Control Language (DCL):**
- GRANT
- REVOKE
  
**Transaction Control Language (TCL):**
- COMMIT
- ROLLBACK
- SAVEPOINT

## Order of execution
- FROM, including JOINs
- WHERE
- GROUP BY
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

## Window Functions
Window fns give the ability to put the values from one row of data into context compared to a group of rows, or partition.

We can answer questions like
- If the dataset were sorted, where would this row land in the results?
- How does a value in this row compare to a value in the prior row?
- How does a value in the current row compare to the average value for its group?

So, window functions **return group aggregate calculations alongside individual row-level information** for items in that group, or partition. 

## Views vs CTEs

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
**To retain the duplicate rows**
```SQL:
  SELECT City FROM Customers
  UNION ALL
  SELECT City FROM Suppliers
  ORDER BY City;
```
