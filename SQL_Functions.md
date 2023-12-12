# SQL STRING Functions

LENGTH(column_name) => Length of string

REPLACE(column_name, 'substr_to_replace', 'replace_by_str') => Replace substring with another string

LEFT(column_name, 3) => Extracting the first 3 letters

RIGHT(column_name, 3) => Extracting the last 3 letters

SUBSTRING(column_name, 2, 3) => Extracting 3 letters from the 2nd position

# SQL DATE Functions

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

# SQL WINDOW Functions

SUM(salary) OVER(ORDER BY salary ASC) AS cumulative_salary

# Used Defined Parameters

**Write a query to extract all the transactions that are made between the dates 2019- 04-24' and '2019-05-24' using user defined parameters "start_date" and "end_date"**

SET @start_date := '2019-04-24';

SET @end_date := '2019-05-24';

SELECT * FROM customer_purchases

WHERE market_date BETWEEN @start_date AND @end_date

ORDER BY market_date;
