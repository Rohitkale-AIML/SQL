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

