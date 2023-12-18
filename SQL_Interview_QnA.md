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
```
