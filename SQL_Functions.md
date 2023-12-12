# SQL STRING Functions

LENGTH(column_name) => Length of string

REPLACE(column_name, 'substr_to_replace', 'replace_by_str') => Replace substring with another string

LEFT(column_name, 3) => Extracting the first 3 letters

RIGHT(column_name, 3) => Extracting the last 3 letters

SUBSTRING(column_name, 2, 3) => Extracting 3 letters from the 2nd position

# SQL DATE Functions

MONTH(date_column) => Gives month number

YEAR(date_column) => Gives year

# SQL WINDOW Functions

SUM(salary) OVER(ORDER BY salary ASC) AS cumulative_salary

# Used Defined Parameters

**Write a query to extract all the transactions that are made between the dates 2019- 04-24' and '2019-05-24' using user defined parameters "start_date" and "end_date"**

SET @start_date := '2019-04-24';

SET @end_date := '2019-05-24';

SELECT * FROM customer_purchases

WHERE market_date BETWEEN @start_date AND @end_date

ORDER BY market_date;