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
