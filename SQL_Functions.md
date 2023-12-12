## SQL STRING Functions

=> LENGTH(column_name) => Length of string

REPLACE(column_name, 'substr_to_replace', 'replace_by_str') => Replace substring with another string

LEFT(column_name, 3) => Extracting the first 3 letters

RIGHT(column_name, 3) => Extracting the last 3 letters

SUBSTRING(column_name, 2, 3) => Extracting 3 letters from the 2nd position

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

## SQL AGGREGATE Functions (GROUP BY Clause)

COUNT(\*), COUNT(1), COUNT(2), COUNT(999) => Gives total rows including NULL & Duplicate

COUNT(column_name) => Gives total rows including Duplicate but excluding NULL

COUNT(DISTINCT column_name) => Gives total rows excluding NULL & Duplicate

SUM(n) => Gives (total rows including NULL & Duplicate) \* n

## SQL WINDOW Functions

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

## Used Defined Parameters

**Write a query to extract all the transactions that are made between the dates 2019- 04-24' and '2019-05-24' using user defined parameters "start_date" and "end_date"**
```SQL:
  SET @start_date := '2019-04-24';
  SET @end_date := '2019-05-24';
  
  SELECT \* 
  FROM customer_purchases
  WHERE market_date BETWEEN @start_date AND @end_date
  ORDER BY market_date;
```
