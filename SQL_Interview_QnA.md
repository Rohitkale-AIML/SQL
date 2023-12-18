**Q: Retrieve the top 5 highest-paid employees for each department, sorted by salary in descending order**
```SQL:
SELECT
	department_id,
	department_name,
	employee_name,
	salary
FROM
	(
		SELECT
			department_id,
			department_name,
			employee_name,
			salary,
			ROW_NUMBER() OVER(PARTITION BY department_id ORDER BY salary DESC) AS rnk
		FROM
			employees
	) x
WHERE
	rnk <= 5;
```

**Q: Write a query to retrieve 3rd highest salary without window function**
```SQL:
SELECT DISTINCT Salary
FROM Employee e1
WHERE 3 = (
    SELECT COUNT(DISTINCT Salary)
    FROM Employee e2
    WHERE e2.Salary >= e1.Salary
);
```

**Q: List the products that have been sold in all cities where the company operates**
```SQL:
SELECT
	product_id,
	product_name
FROM
	products
WHERE
	product_id NOT IN (
	SELECT
		DISTINCT product_id
	FROM
		sales
	WHERE
		city NOT IN (
		SELECT
			DISTINCT city
		FROM
			locations
		)
	);
```

**Q: Calculate the 30-day moving average of sales for each product**
```SQL:
SELECT
	product_id,
	sales_date,
	sales_amount,
	AVG(sales_amount) OVER(PARTITION BY product_id ORDER BY sales_date 
	                       RANGE BETWEEN INTERVAL '30 days' PRECEDING AND CURRENT ROW) AS moving_avg_30days
FROM
	sales;
```

**Q: List the departments where the average salary is higher than the company's overall average salary**
```SQL:
SELECT
	department
FROM
	employees
GROUP BY 
	department
HAVING
	AVG(salary) > (SELECT AVG(salary) FROM employees);
```

**Q: How to create empty table with same structure as another table?**
```SQL:
CREATE TABLE new_table AS(
SELECT * FROM source_table WHERE 1 = 0);
```

**Q: Identify how many cars, Motorcycles, trains and ships are available in the inventory? Treat all type of cars as just "Cars"**
```SQL:
-- Solution 1:
SELECT product_line, count(1)
FROM (
        SELECT CASE WHEN product_line IN ('Classic Cars', 'Vintage Cars') THEN 'Cars'
                    ELSE product_line
               END AS product_line
        FROM products
        WHERE product_line IN ('Classic Cars', 'Vintage Cars', 'Motorcycles', 'Trains', 'Ships')
    )
GROUP BY product_line;


-- Solution 2:
SELECT CASE WHEN product_line IN ('Classic Cars', 'Vintage Cars') THEN 'Cars'
            ELSE product_line
       END AS product_line
, count(1)
FROM products
WHERE product_line IN ('Classic Cars', 'Vintage Cars', 'Motorcycles', 'Trains', 'Ships')
GROUP BY CASE WHEN product_line IN ('Classic Cars', 'Vintage Cars') THEN 'Cars'
              ELSE product_line
         END ;

SELECT CASE WHEN product_line IN ('Classic Cars', 'Vintage Cars') THEN 'Cars'
            ELSE product_line
       END AS new_product_line
, count(1)
FROM products
WHERE product_line IN ('Classic Cars', 'Vintage Cars', 'Motorcycles', 'Trains', 'Ships')
GROUP BY new_product_line;


-- Solution 3:
SELECT product_line, count(1)
FROM products
WHERE product_line in ('Motorcycles', 'Trains', 'Ships')
GROUP BY product_line
UNION
SELECT 'Cars' as prod, count(1)
FROM products
WHERE product_line IN ('Classic Cars', 'Vintage Cars');
```

**Q: Write a query to identify the duplicate records?**
```SQL:
SELECT first_name, last_name, count(1)
FROM artists
GROUP BY first_name, last_name
HAVING count(1) > 1;
```

**Q:How would you delete the duplicate records in a table?**
```SQL:
-- in PostgreSQL
SELECT a.*, a.ctid FROM artists a;
-- ctid works like row number

-- If single duplicate records present
DELETE FROM artists
WHERE ctid IN (
               SELECT MAX(ctid)
               FROM artists
               GROUP BY first_name, last_name
               HAVING count(1) > 1);

-- If multiple duplicate records present
DELETE FROM artists
WHERE ctid NOT IN (
                   SELECT MIN(ctid)
                   FROM artists
                   GROUP BY first_name, last_name);

-- If the table already has a unique column (like id in this case)
DELETE FROM artists
WHERE id NOT IN (
                 SELECT MIN(id)
                 FROM artists
                 GROUP BY first_name, last_name);

-- Using window function
DELETE FROM artists
WHERE id IN ( SELECT id 
              FROM (
                     SELECT *,
                            ROW_NUMBER() OVER(PARTITION BY first_name, last_name )  AS rn
                     FROM artists) x
              WHERE rn > 1);

-- In MySQL:
-- Using self join
DELETE FROM artists
WHERE id IN (
             SELECT L.id FROM artists AS L
	     INNER JOIN artists AS R
	     ON L.id > R.id
		AND L.first_name = R.first_name
		AND L.last_name = R.last_name);
```

**Q: Find how much each artist made from sales. And how many paintings did they sell?**
```SQL:
SELECT
	s.artist_id, 
	CONCAT(a.first_name, ' ',a.last_name) AS artist_name, 
	count(painting_id) AS painitngs_sold, 
	sum(sales_price) AS total_earnings
FROM sales AS s
INNER JOIN (SELECT id, first_name, last_name FROM artists) a --efficient way of join with required columns only
ON s.artist_id = a.id
GROUP BY s.artist_id, a.first_name, a.last_name
ORDER BY s.artist_id;
```

**Q: Display all the available paintings and all the artist. If a painting was sold then mark them as "Sold" and if more than 1 painting of an artist was sold then display a "\**" beside their name.**
```SQL:
SELECT 
    p.name as painting_name, 
    CONCAT(a.first_name, ' ', a.last_name, 
           CASE WHEN x.no_of_paintings > 1 THEN ' **' END) as artist_name, 
    CASE 
        WHEN s.id IS NOT NULL THEN 'SOLD' 
    END AS sold_or_not
FROM paintings p
FULL JOIN artists a 
     ON a.id = p.artist_id
LEFT JOIN sales s 
     ON s.painting_id = p.id
LEFT JOIN (SELECT artist_id, COUNT(*) AS no_of_paintings
           FROM sales 
           GROUP BY artist_id) x 
     ON x.artist_id = a.id;

-- common table expression
WITH cte AS
	(SELECT p.id AS painting_id,
                a.id AS artist_id,
                p.name AS painting_name,
                CONCAT(a.first_name, ' ', a.last_name) AS artist_name
	FROM paintings p
	LEFT JOIN artists a
             ON a.id = p.artist_id
	UNION
	SELECT p.id AS painting_id,
               a.id AS artist_id,
               p.name AS painting_name,
               concat(a.first_name, ' ', a.last_name) AS artist_name
	from paintings p
	RIGHT JOIN artists a
              ON a.id = p.artist_id)
SELECT painting_name,
       CONCAT(cte.artist_name, CASE WHEN x.no_of_paintings > 1 THEN ' **' END ) AS artist_name,
       CASE WHEN s.id IS NOT NULL THEN 'SOLD' END AS sold_or_not
FROM cte
LEFT JOIN sales s
     ON s.painting_id = cte.painting_id
LEFT JOIN (SELECT artist_id, count(*) AS no_of_paintings
           FROM sales
           GROUP BY artist_id) x
     ON x.artist_id = cte.artist_id;
```

**Q: Generate the following two result sets:**
1. Query an alphabetically ordered list of all names in OCCUPATIONS, immediately followed by the first letter of each profession as a parenthetical (i.e.: enclosed in parentheses). For example: An ActorName(A), A DoctorName(D), A ProfessorName(P), and A SingerName(S).

2. Query the number of ocurrences of each occupation in OCCUPATIONS. Sort the occurrences in ascending order, and output them in the following format: There are a total of [occupation_count] [occupation]s.

where [occupation_count] is the number of occurrences of an occupation in OCCUPATIONS and [occupation] is the lowercase occupation name. If more than one Occupation has the same [occupation_count], they should be ordered alphabetically.
```SQL:
(SELECT
    CONCAT(name, '(', UPPER(LEFT(occupation,1)), ')') AS title
FROM
    occupations    
)
UNION
(SELECT
    CONCAT('There are a total of ', COUNT(1), ' ', LOWER(occupation), 's.') AS title
FROM 
    occupations
GROUP BY
    occupation
)
ORDER BY 1
```

**Q: Find the most profitable orders. Most profitable orders are those whose sale price exceeded the average sale price for each city and whose deal size was not small?**
```SQL:
-- Solution 1: Using WITH clause
WITH avg_sales_per_city AS
    (SELECT c.city, AVG(sales) AS avg_sales
     FROM sales_order s
     JOIN customers c ON c.customer_id = s.customer
     GROUP BY c.city)
SELECT s.*
FROM sales_order s
INNER JOIN customers c ON c.customer_id = s.customer
INNER JOIN avg_sales_per_city av ON av.city = c.city
WHERE s.sales > av.avg_sales
AND s.deal_size <> 'Small';

-- Solution 2: Using Correlated subquery
SELECT s.*
FROM sales_order s
INNER JOIN customers c 
      ON c.customer_id = s.customer
WHERE s.sales > (SELECT AVG(sales)
                 FROM sales_order s2
                 INNER JOIN customers c2 ON c2.customer_id = s2.customer
                 WHERE c2.city = c.city
                 GROUP BY c.city)
AND s.deal_size <> 'Small';
```

**Q: Find the difference in average sales for each month of 2003 and 2004**
```SQL:
-- Solution 1: Using multiple WITH clause temp tables.
WITH yr_2003 AS
        (SELECT year_id, 
                TO_CHAR(order_date, 'MON') AS mon, 
                AVG(sales) AS avg_sales
         FROM sales_order s
         WHERE year_id = 2003
         GROUP BY year_id, TO_CHAR(order_date, 'MON')),
     yr_2004 AS
        (SELECT year_id, 
                TO_CHAR(order_date, 'MON') AS mon, 
                AVG(sales) AS avg_sales
         FROM sales_order s
         WHERE year_id = 2004
         GROUP BY year_id, TO_CHAR(order_date, 'MON'))
SELECT Y03.mon,
       ROUND(ABS(Y03.avg_sales - Y04.avg_sales)::DECIMAL,2) AS diff
FROM yr_2003 Y03
INNER JOIN yr_2004 Y04 ON Y03.mon = Y04.mon;

-- Solution 2: using SELF join.
WITH cte AS
    (SELECT year_id, 
            TO_CHAR(order_date, 'MON') AS mon, 
            AVG(sales) AS avg_sales
     FROM sales_order s
     WHERE year_id in (2003, 2004)
     GROUP BY year_id, TO_CHAR(order_date, 'MON'))
SELECT Y03.mon, ROUND(ABS(Y03.avg_sales - Y04.avg_sales)::DECIMAL,2) AS diff
FROM cte AS Y03
INNER JOIN cte AS Y04
     ON Y03.mon = Y04.mon
WHERE Y03.year_id = 2003
AND Y04.year_id = 2004;
```

**Q: Find the popularity percentage for each user on Meta/Facebook. The popularity percentage is defined as the total number of friends the user has divided by the total number of users on the platform, then converted into a percentage by multiplying by 100. Output each user along with their popularity percentage. Order records in ascending order by user id.**
```SQL:
SELECT * FROM facebook_friends;
```

![facebook_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/facebook_friends.png?raw=true)

-- a) Find how many friends each user has? 

-- b) Find total users in FB

-- c) Find PP = a / b

```SQL:
-- b)
SELECT user1 FROM facebook_friends
UNION
SELECT user2 FROM facebook_friends
```
```sql:
-- a)
WITH all_users AS
    (SELECT user1 AS users FROM facebook_friends
     UNION ALL
     SELECT user2 AS users FROM facebook_friends)
SELECT users, COUNT(1) AS no_of_frnds
FROM all_users
GROUP BY users
ORDER BY 1;
```
```sql:
-- c)
WITH all_users AS
        (SELECT user1 AS users FROM facebook_friends
         UNION ALL
         SELECT user2 AS users FROM facebook_friends),
     user_frnds AS
        (SELECT users, COUNT(1) AS no_of_frnds
         FROM all_users
         GROUP BY users),
     unq_users AS
        (SELECT COUNT(DISTINCT users) AS total_users
         FROM all_users) -- THIS HAS ONLY 1 RECORD
SELECT a.users,
ROUND(((a.no_of_frnds::DECIMAL / b.total_users::DECIMAL) * 100),2) AS popularity_percentage
FROM user_frnds a
CROSS JOIN unq_users b -- JOIN M RECORDS WITH 1 RECORDS
ORDER BY 1;
```

**Q:Find the number of emails received by each user under each built-in email label. The email labels are: 'Promotion', 'Social', and 'Shopping'. Output the user along with the number of promotion, social, and shopping mails count**
```SQL:
select * from google_gmail_emails;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/ggoogle_gmail_emails.png?raw=true)

```SQL:
select * from google_gmail_labels;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/google_gmail_labels.png?raw=true)

```SQL:
SELECT to_user, 
       SUM(CASE WHEN label = 'Promotion' THEN 1 ELSE 0 END) AS Promotion, 
       SUM(CASE WHEN label = 'Social' THEN 1 ELSE 0 END) AS Social, 
       SUM(CASE WHEN label = 'Shopping' THEN 1 ELSE 0 END) AS Shopping
FROM google_gmail_emails AS E
INNER JOIN google_gmail_labels L 
       ON L.email_id = E.id
WHERE L.label IN ('Promotion', 'Social', 'Shopping')
group by to_user;
```

**Q: Find the date with the highest total energy consumption from the Meta/Facebook data centers. Output the date along with the total energy consumption across all data centers. If there are multiple days with same highest energy consumption then display both dates.**
```SQL:
SELECT * FROM fb_eu_energy;
```
![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/fb_eu_energy.png?raw=true)

```SQL:
SELECT * FROM fb_asia_energy;
```
![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/fb_asia_energy.png?raw=true)

```SQL:
SELECT * FROM fb_na_energy;
```
![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/fb_na_energy.png?raw=true)

```SQL:
-- SOLUTION 01
WITH all_energy AS
        (SELECT * FROM fb_eu_energy
        UNION ALL
        SELECT * FROM fb_asia_energy
        UNION ALL
        SELECT * FROM fb_na_energy),
    agg_energy AS
        (SELECT date, SUM(consumption) AS tot_consumption
         FROM all_energy
         GROUP BY date),
    cte AS
        (SELECT *, RANK() OVER(ORDER BY tot_consumption DESC) AS rnk
         FROM agg_energy)
SELECT date, tot_consumption
FROM cte
WHERE rnk = 1;

-- SOLUTION 02
SELECT * FROM (
    SELECT *, RANK() OVER(ORDER BY tot_consumption DESC) AS rnk
    FROM (SELECT date, SUM(consumption) AS tot_consumption
          FROM (SELECT * FROM fb_eu_energy
                UNION ALL
                SELECT * FROM fb_asia_energy
                UNION ALL
                SELECT * FROM fb_na_energy)X
          GROUP BY date)Y
) xx
WHERE rnk = 1;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/total_consumption_solu.png?raw=true)

**Q: From the students table, write a SQL query to interchange the adjacent student names. Note: If there are no adjacent student then the student name should stay the same.**
```SQL:
SELECT * FROM students;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/students.png?raw=true)

```SQL:
SELECT id, 
       CASE WHEN id%2 <> 0 THEN LEAD(student_name,1,student_name) OVER(ORDER BY id)
            WHEN id%2 = 0 THEN LAG(student_name, 1, 'NO RECORD FOUND') OVER(ORDER BY id)
       END AS new_student_name
FROM students ;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/students_result.png?raw=true)

**Q: Find the best selling item for each month (no need to separate months by year) where the biggest total invoice was paid. The best selling item is calculated using the formula (unitprice * quantity). Output the description of the item along with the amount paid.**
```SQL:
SELECT * FROM online_retail;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/online_retail.png?raw=true)

```SQL:
WITH cte AS
    (SELECT description AS item, 
	        TO_CHAR(invoicedate, 'MM') AS mon_id , 
	        TO_CHAR(invoicedate, 'MON') AS mon, 
	        ROUND(SUM(quantity * unitprice)::DECIMAL,2) AS tot_sales, 
	        RANK() OVER(PARTITION BY TO_CHAR(invoicedate, 'MM') ORDER BY SUM(quantity * unitprice) DESC) AS rnk
     FROM online_retail
     GROUP BY description, TO_CHAR(invoicedate, 'MM'), TO_CHAR(invoicedate, 'MON'))
SELECT item, mon, tot_sales
FROM cte
WHERE rnk = 1;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/online_retail_result.png?raw=true)

**Q: Write query to display the most, least & Second most expensive product under each category (corresponding to each record)**
```SQL:
SELECT * FROM product;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/products.png?raw=true)

```SQL:

SELECT *,
       FIRST_VALUE(product_name) OVER(PARTITION BY product_category ORDER BY price DESC
                       		      RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS most_exp_prod,
       LAST_VALUE(product_name) OVER(PARTITION BY product_category ORDER BY price DESC
                                     RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS least_exp_prod,
       NTH_VALUE(product_name, 2) OVER(PARTITION BY product_category ORDER BY price DESC
                                       RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS second_most_exp_prod
FROM product;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/products_result.png?raw=true)

**Q: Make a pivot table to find the highest payment in each year for each employee. Output payment details along with the corresponding employee name.**
```SQL:
SELECT * FROM sf_public_salaries
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/pivot_table.png?raw=true)

```SQL:
-- syntax for crosstab
SELECT *
FROM CROSSTAB('base query', 'list of columns ') -- second argument is optional
     AS RESULT (column_name1 data_type, ...);
	 
-- Solution:
CREATE EXTENSION tablefunc; -- crosstab is part of this extension..

SELECT EMPLOYEENAME, 
       COALESCE(PAY_2011, 0) AS PAY_2011, 
	   COALESCE(PAY_2012, 0) AS PAY_2012, 
	   COALESCE(PAY_2013, 0) AS PAY_2013, 
	   COALESCE(PAY_2014, 0) AS PAY_2014
FROM crosstab(' SELECT employeename, year, MAX(totalpay) AS max_payment
                FROM sf_public_salaries
                GROUP BY employeename, year
                ORDER BY employeename, year '
             , 'SELECT DISTINCT year FROM sf_public_salaries')
    AS RESULT (employeename VARCHAR, PAY_2011 FLOAT, PAY_2012 FLOAT, PAY_2013 FLOAT, PAY_2014 FLOAT ); 
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/pivot_table_result.png?raw=true)

**RECURSIVE SQL QUERIES**
```SQL:
-- Syntax:

WITH RECURSIVE cte AS
    (base query -- the starting point
     UNION / UNION ALL
     recursive part of the query -- which use the result from cte
     include a termination condition
    )
SELECT *
FROM cte;
```

**Q1: Display number from 1 to 10 without using any in built functions.**
```SQL:
WITH RECURSIVE cte AS
    (SELECT 1 AS n
     UNION
     SELECT (n + 1) AS n
     FROM cte
     WHERE n < 10)
SELECT *
FROM cte;
```

![sample_image](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/recursive.png?raw=true)

**Q: Write a query to segregate all the expensive phones, mid range phones and the cheaper phones.**
```SQL:
SELECT x.product_name,
	   CASE WHEN x.buckets = 1 THEN 'Expensive Phones'
		    WHEN x.buckets = 2 THEN 'Mid Range Phones'
		    WHEN x.buckets = 3 THEN 'Cheaper Phones' 
	   END AS Phone_Category
FROM (
    SELECT *,
    NTILE(3) OVER(ORDER BY price DESC) AS buckets
    FROM product
    WHERE product_category = 'Phone') x;
```

![SAMPLE_IMAGE](https://github.com/Rohitkale-AIML/SQL/blob/main/ERD-images/NTILE.png?raw=true)
