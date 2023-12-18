## SQL STRING Functions

LOWER(column_name) => Lower case string

UPPER(column_name) => Upper case string

LENGTH(column_name) => Length of string

REPLACE(column_name, 'old_str_to_replace', 'replace_by_new_str') => Replace old string with new string

LEFT(column_name, length) => Extracting the first 'length' letters

RIGHT(column_name, length) => Extracting the last 'length' letters

SUBSTRING(column_name, start, length) => Extracting length letters from 'start' position

CHARINDEX(substr, column_name, start) => Find location of 'substr' in given column_name from 'start' position

CONCAT(column_name, ' ', column_name) => concatenate multiple column values into one single value

TRIM(column_name) => Remove white spaces from both ends

LRIM(column_name) => Remove white spaces from front/left

RRIM(column_name) => Remove white spaces from back/right

## SQL DATE Functions

CURDATE() => Gives current Date YYYY-MM-DD

CURTIME() => Gives current Time HH:MM:SS (24 hour)

DAY(date_column) OR EXTRACT(DAY FROM date_column) => Gives day number

MONTH(date_column) OR EXTRACT(MONTH FROM date_column) => Gives month number 

YEAR(date_column) OR EXTRACT(YEAR FROM date_column) => Gives year

HOUR(date_time_column) OR EXTRACT(HOUR FROM date_column) => Gives hour of day

MINUTE(date_time_column) OR EXTRACT(MINUTE FROM date_column) => Gives minute of time

WEEKDAY(date_column) => Gives weekday number

DAYNAME(date_column) => Gives dayname

MONTHNAME(date_column) => Gives name of month

DATE(date_time_column) => Extracting the entire date part

TIME(date_time_column) => Extracting the entire time part

DATE_ADD(date_time_column, INTERVAL 30 MINUTE) => Add 30 mins

DATE_SUB(date_time_column, INTERVAL 15 DAY) => Subtract 15 days

DATEDIFF('High_date', 'Low_date') => number of days difference betwen two dates

TIMESTAMPDIFF(HOUR, start_datetime, end_datetime) => hour duration between two timestamps

TIMESTAMPDIFF(MINUTE, start_datetime, end_datetime) => minute duration between two timestamps

STR_TO_DATE(string_column, "%Y-%m-%d %h:%i:%p") AS market_start_datetime => String to Date_Time Format

## SQL MATH Functions

SELECT MOD(13, 2); => Get the remainder of a division

SELECT ROUND(1234.56789, 3); => Round a number to three decimal places

SELECT CEIL(-13.9); -- result: -13 => The CEIL(x) function returns the smallest integer not less than x

SELECT FLOOR(-13.2); -- result: -14 => The FLOOR(x) function returns the greatest integer not greater than x

SELECT TRUNC/TRUNCATE(-13.5); -- result: -13 => To round towards 0 irrespective of the sign of a number

SELECT ABS(-12);

SELECT SQRT(9);

## USEFUL FUNCTIONS
**COALESCE(x, y, ...)**
- To replace NULL in a query with something meaningful
- The COALESCE() function takes any number of arguments and returns the value of the first argument that isn't NULL.
- COALESCE(column_name1, column_name2, column_name3, 'value missing')

**NULLIF(x, y)**
- To save yourself from division by 0 errors
- The NULLIF(x, y) function will return NULL if x is the same as y, else it will return the x value.
- NULLIF(column_name, 0)

**SELECT CAST('2021-12-31' AS date);**

**IFNULL(column_name, 'NA')**
- It will return 'NA' if value in column is NULL

## SQL AGGREGATE Functions (GROUP BY Clause)

COUNT(\*), COUNT(1), COUNT(2), COUNT(999), COUNT('xyz') => Gives total rows including NULL & Duplicate

COUNT(column_name) => Gives total rows including Duplicate but excluding NULL

COUNT(DISTINCT column_name) => Gives total rows excluding NULL & Duplicate

SUM(n) => Gives (total rows including NULL & Duplicate) \* n

MIN()

MAX()

AVG()

## SQL WINDOW Functions

- Unlike aggregate functions, window functions do not collapse rows.
- You can use window functions in SELECT and ORDER BY
- The window frame is evaluated separately within each partition. => ROWS | RANGE BETWEEN lower_bound AND upper_bound

**DEFAULT WINDOW FRAME:**
- If ORDER BY is specified, then the frame is RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW.
- Without ORDER BY, the frame specification is ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING.

**RANKING FUNCTIONS:**
- ROW_NUMBER() − unique number for each row within partition, with different numbers for tied values
- RANK() − ranking within partition, with gaps and same ranking for tied values
- DENSE_RANK() − ranking within partition, with no gaps and same ranking for tied values

**ANALYTIC FUNCTIONS:**
- LEAD(expr, offset, default) − the value for the row offset rows after the current; offset and default are optional; default values: offset = 1, default = NULL
- LAG(expr, offset, default) − the value for the row offset rows before the current; offset and default are optional; default values: offset = 1, default = NULL
- NTILE(n) − divide rows within a partition as equally as possible into n groups, and assign each row its group number
- FIRST_VALUE(expr) − the value for the first row within the window frame 
- LAST_VALUE(expr) − the value for the last row within the window frame
- NTH_VALUE(expr, n) − the value for the n-th row within the window frame; n must be an integer
 
SUM(salary) OVER(ORDER BY salary ASC) AS cumulative_salary

SUM(sale) OVER(ORDER BY date ASC RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_sales 

- Above two SQL statements are equvivalent to get cumulative SUM

SUM(quantity \* cost_to_customer_per_qty) OVER(PARTITION BY customer_id ORDER BY market_date) AS customer_spend_running_total

- Here order by clause is very important to get running total without that it will give all record total for each customer_id and not running/cumulative total

AVG(sale) OVER(ORDER BY date ASC RANGE BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS range_moving_avg

AVG(sale) OVER(ORDER BY date ASC ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS rows_moving_avg

- Here RANGE will specifies the window frame considering the values in the date column and ROWS will specifies the window frame considering the physical order of rows in the result set.

NTILE(10) OVER(ORDER BY original_price DESC) AS price_ntile

- NTILE(n) is a window function that distributes rows of an ordered partition into a specified number (n) of approximately equal groups, or buckets. It assigns each group a bucket number starting from one.

## User Defined Parameters

**Write a query to extract all the transactions that are made between the dates 2019- 04-24' and '2019-05-24' using user defined parameters "start_date" and "end_date"**
```SQL:
SET @start_date := '2019-04-24';
SET @end_date := '2019-05-24';

SELECT \* 
FROM customer_purchases
WHERE market_date BETWEEN @start_date AND @end_date
ORDER BY market_date;
```
