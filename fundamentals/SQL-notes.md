# SQL

Created: March 29, 2023 1:56 PM <br/>
Domain: sql<br/>
Type: sql-notes<br/>

# Intro

For each database:

- Tables: where we stored the data
- Views: virtual tables, we can combine data from multiple tables and put them into a view
- Stored Procedures
- Functions

Multiple tables related to each other using a relationship.

# Operations

## SELECT

- could use * to select all columns
- could also select specific columns from the table

```sql
 SELECT DISTINCT state FROM customers -- show distinct elements
```

```sql
SELECT 
	last_name, 
	first_name, 
	points, 
  (points * 10 + 100) AS discount_factor -- use AS to name the new column
FROM customers
```

## WHERE

- filter data
    - '>'
    - '> ='
    - '<'
    - '< ='
    - '='
    - '! = OR < >'
- Combine multiple search conditions and filter data
    - AND
    - OR: combine multiple conditions
    - NOT
        
        ```sql
        WHERE NOT (birth_date >= '1990-01-01' OR points > 1000)
        ```
        
        could be rewritten as:
        
        ```sql
        WHERE birth_date < '1990-01-01' AND points <= 1000
        ```
        
    
    NOTE: orders: AND is always evaluated first
    
    - can always change the order by parenthesis

### IN

Can replace OR operator

```sql
WHERE state = 'VA' OR state = 'GA' OR state = 'FL’
```

```sql
WHERE state IN ('VA', 'FL', 'GA')
```

### BETWEEN

```sql
WHERE points BETWEEN 1000 AND 3000
```

- The boundary are included

### LIKE

- retrieve rows match a specific string pattern

%

- use `%` to represent any string (could represent single or multiple characters)
- `%` can at anywhere
    - `%b%’` : can have any number before and after b

_

- represents a single character

### REGEXP

- regular expression: allows to search for more complex patterns

```sql
WHERE last_name LIKE '%field%’
```

```sql
WHERE last_name REGEXP 'field'
```

- put `^` before the string:
    - **^ beginning**
    - present the beginning of the string
    - eg.
    
    ```sql
    WHERE last_name REGEXP '^field'
    ```
    
    - the searched string must start with field
- put `$` before the string:
    - **$ end**
    - present the end of the string
    - eg.
    
    ```sql
    WHERE last_name REGEXP 'field$'
    ```
    
    - the searched string must end with field
- use `|` to present multiple search patterns
    - **| logical or**
    
    ```sql
    WHERE last_name REGEXP 'field|mac|rose’
    ```
    
- use `[]`
    - **[abcd]**
    - **[a-f]**
    
    ```sql
    WHERE last_name REGEXP '[gim]e’
    ```
    
    - present any last_name with `ge` or `ie` or `me`
    - supply a range of characters
        - `[a-h]e`: range from a to h before e

### IS NULL

```sql
WHERE phone IS NULL
```

```sql
WHERE phone IS NOT NULL
```

## ORDER BY

Every table should have a primary column, which is ordered by default.

```sql
ORDER BY first_name DESC -- sort by descending order
```

## LIMIT

```sql
LIMIT 3 -- return the first 3 rows
```

```sql
LIMIT 6, 3 -- 6 is an offset (to skip the first 6 record), then pick 3 record
```

- Always at the end of the query

# JOINs

## INNER JOIN

Join the order table and customer table, and make sure the customer_id column in the order table == customer_id column in the customer table.

```sql
SELECT * 
FROM orders
JOIN customers 
	ON orders.customer_id = customers.customer_id
```

```sql
SELECT order_id, **orders.customer_id**, first_name, last_name -- the customer_id appears in both table, need to add a prefix
FROM orders
JOIN customers 
	ON orders.customer_id = customers.customer_id
```

- Use alias
- if use alias at one place, must use alias at all other places, or it would raise error.

```sql
SELECT order_id, **o.customer_id**, first_name, last_name
FROM orders **o**
JOIN customers **c**
	ON **o**.customer_id = **c.**customer_id
-- can make the expression clear 
```

### Join across database

- only have to prefix the data that are not in the current database

```sql
SELECT * 
FROM order_items oi
JOIN **sql_inventory**.products p 
	ON oi.product_id = p.product_id
```

### Self Joins

- join a table with itself
- Use different alias, and use a different prefix for each column

```sql
SELECT e.employee_id, e.first_name, m.first_name AS manager -- a different name to differ columns
FROM employees e
JOIN employees m -- short for managers, they are actually the same table
	ON e.reports_to = m.employee_id
```

### Joining multiple tables

```sql
USE sql_store;

SELECT *
FROM orders o
JOIN customers c
	ON o.customer_id = c.customer_id
JOIN order_statuses os
	ON o.status = os.order_status_id
```

### Compound Join Conditions

- join table based on multiple columns (use AND)

```sql
SELECT *
FROM order_items oi
JOIN order_item_notes oin
	ON oi.order_id = oin.order_id 
    AND oi.product_id = oin.product_id
```

### Implicit Join Syntax

- explicit join syntax: (always better to use)

```sql
SELECT *
FROM orders o
JOIN customers c
	ON o.customer_id = c.customer_id
```

- the same as the following implicit join syntax:

```sql
SELECT *
FROM orders o, customers c
WHERE o.customer_id = c.customer_id
```

## OUTER JOIN

### LEFT JOIN & RIGHT JOIN

- all the records on the left table will be returned whether the ON condition is true or not
- keep in mind the order of tables
- best to use left join rather than right join

```sql
SELECT 
	c.customer_id,
    c.first_name,
    o.order_id
FROM customers c
**LEFT JOIN** orders o
	ON c.customer_id = o.customer_id
ORDER BY c.customer_id
```

### Outer join between multiple tables

```sql
SELECT 
	c.customer_id,
    c.first_name,
    o.order_id,
    sh.name AS shipper
FROM customers c
LEFT JOIN orders o
	ON c.customer_id = o.customer_id
LEFT JOIN shippers sh
	ON o.shipper_id = sh.shipper_id
ORDER BY c.customer_id
```

### Self outer joins

```sql
USE sql_hr;

SELECT 
	e.employee_id,
    e.first_name,
    m.first_name AS manager
FROM employees e
LEFT JOIN employees m
	ON e.reports_to = m.employee_id
```

### `Using` Clause

- used while the column name is the same across all tables

```sql
USE sql_store;

SELECT 
	o.order_id,
    c.first_name
FROM orders o
JOIN customers c
	-- ON o.customer_id = c.customer_id
    USING (customer_id)
```

- join multiple columns
    
    ```sql
    USING (customer_id, product_id)
    ```
    

### Natural join (pure join, not recommended)

- join tables based on common columns (column with same name)

### Cross join

- join every record with first table with every record with second table

```sql
SELECT 
	sh.name AS shipper,
    p.name AS product
-- FROM shippers sh, products p > implicit syntax

FROM shippers sh
CROSS JOIN products p
ORDER BY shipper
```

### Union

- combine results from multiple queries

```sql
SELECT 
	order_id,
    order_date,
    'Active' as status
FROM orders
WHERE order_date >= '2019-01-01'
**UNION**
SELECT 
	order_id,
    order_date,
    'Archive' as status
FROM orders
WHERE order_date < '2019-01-01'
```

# Insert/ Update/ Delete Data

## Column attributes

`VARCHAR(50)`: variable characters with a maximum length of 50 

- recommended

`CHAR(50)`: if the length of the string is 5, would have another 45 spaces to fill in (what a waste)

`PK`: primary key

`NN`: not null 

- if they are selected, those columns must be filled with a value

`AI`: auto-increment, often use with PK column

`Default / Expression`: 

- if it’s NULL, mySQL will simply fill it with a NULL value
- if it’s ‘0’, ‘0’ will be used

## Insert Rows

### Insert a single row

```sql
INSERT INTO customers
VALUES (DEFAULT, 
		'John', 
		'Smith', 
        '1990-01-01', 
        NULL,
        'address',
        'city',
        'CA',
        DEFAULT)
```

```sql
-- in this way, the default values do not need to be provided
-- and the information could be typed in any order
INSERT INTO customers (
	first_name,
    last_name,
    birth_date,
    address,
    city,
    state)
    
VALUES (
		'John', 
		'Smith', 
        '1990-01-01', 
        'address',
        'city',
        'CA')
```

### Insert multiple rows

```sql
INSERT INTO shippers (
	name)

VALUES ('shipper1'),
	     ('shipper2'),
       ('shipper3')
```

### Insert hierarchy rows

```sql
INSERT INTO orders (customer_id, order_date, status)
VALUES (1, '1990-01-02', 1);

-- **SELECT LAST_INSERT_ID()**; -- get the ID of the newly inserted record
INSERT INTO order_items 
VALUES (LAST_INSERT_ID(), 1, 1, 2.95),
	   (LAST_INSERT_ID(), 2, 1, 3.95)
```

## Creating a Copy of a Table

- quickly copy data from one table to another

```sql
**CREATE TABLE orders_archived AS**
SELECT *  -- sub-query
FROM orders

-- TBN, mySQL will ignore some attributes, 
-- such as PK (primary key), and AI (auto-increment))
```

- use the power of INSERT statement

```sql
INSERT INTO orders_archived
SELECT * 
FROM orders
WHERE order_date < '2019-01-01'
```

Exercise

```sql
CREATE TABLE invoices_archives
SELECT 
	i.invoice_id, 
    i.number, 
    c.name,
    i.invoice_total,
    i.invoice_date,
    i.due_date,
    i.payment_date
FROM invoices i
LEFT JOIN clients c
	ON i.client_id = c.client_id
WHERE i.payment_date is not NULL
```

## Updating Data

### Updating a single row

```sql
UPDATE invoices
SET payment_total = 10, payment_date = '2019-03-01'
WHERE invoice_id = 1
```

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5, 
	payment_date = due_date
WHERE invoice_id = 3
```

### Updating multiple rows

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5, 
	payment_date = due_date
WHERE client_id IN (3, 4)
```

### Using ***subqueries*** in Updates

- Subqueries is a SELECT statement in another SQL statement

```sql
UPDATE invoices
SET
	payment_total = invoice_total * 0.5,
    payment_date = due_date
WHERE client_id IN
			(SELECT client_id
			FROM clients
			WHERE state IN ('CA', 'NY'))
```

## Deleting Data

```sql
DELETE FROM invoices
WHERE client_id =(
	SELECT client_id
	FROM clients
	WHERE name = 'Myworks')
```

## Restoring Databases

file → open SQL scripts → select the “create-database.sql” → open and run the file

# Summarize data

## Aggregate functions

- only apply to non-NULL values

```sql
MAX()
MIN()
AVG()
SUM()
COUNT()
```

```sql
SELECT 
	MAX(invoice_total) AS highest,
	MIN(invoice_total) AS lowest,
	AVG(invoice_total) AS average,
	SUM(invoice_total * 1.1) AS total,
	COUNT(**DISTINCT** client_id) AS total_client -- only get the distinct values
FROM invoices
WHERE invoice_date >= '2019-07-01'
```

- to includes the NULL values

```sql
COUNT(*) AS total_records -- return the total records with NULL values
```

Exercise:

- Remember we could use ` UNION ` to combine several results

```sql
SELECT 
	'First half of 2019' AS date_range,
    SUM(invoice_total) AS total_sales,
    SUM(payment_total) AS total_payments,
    SUM(invoice_total - payment_total) AS 'what we expect'
FROM invoices
WHERE invoice_date BETWEEN '2019-01-01' AND '2019-06-30'

UNION

SELECT 
	'Second half of 2019' AS date_range,
    SUM(invoice_total) AS total_sales,
    SUM(payment_total) AS total_payments,
    SUM(invoice_total - payment_total) AS 'what we expect'
FROM invoices
WHERE invoice_date BETWEEN '2019-07-01' AND '2019-12-31'

UNION

SELECT 
	'total' AS date_range,
    SUM(invoice_total) AS total_sales,
    SUM(payment_total) AS total_payments,
    SUM(invoice_total - payment_total) AS 'what we expect'
FROM invoices
```

## Group By

- should group data by all the columns in the `SELECT` clause

```sql
SELECT 
	client_id,
	SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id -- see the total sale number for each client 
```

- be careful about the order of WHERE → GROUP BY → ORDER BY

```sql
SELECT 
	client_id,
	SUM(invoice_total) AS total_sales
FROM invoices
WHERE invoice_date >= '2019-07-01'
GROUP BY client_id
ORDER BY total_sales DESC
```

```sql
SELECT
	  date,
    pm.name AS payment_method,
    SUM(amount) AS total_payments
FROM payments p
JOIN payment_methods pm
	ON p.payment_method = pm.payment_method_id 
GROUP BY date, payment_method
ORDER BY date
```

## Having

- using HAVING clause to filter data after grouping the rows
- can have one or more conditions
- column we used in the having clause, have to be a part of the `select` clause
    - (can reference any column in WHERE clause)

```sql
SELECT 
	client_id,
    SUM(invoice_total) AS total_sales,
    COUNT(*) AS number_of_invoices
FROM invoices
GROUP BY client_id
HAVING total_sales > 500 AND number_of_invoices > 5
```

```sql
SELECT
	c.first_name,
	o.customer_id,
	-- o.order_id, 
    c.state,
    SUM(oi.quantity * oi.unit_price) AS total_spent
FROM orders o
JOIN order_items oi USING (order_id)
JOIN customers c USING (customer_id)
WHERE state = 'VA'
GROUP BY customer_id, state, first_name
HAVING total_spent > 100
```

## WITH ROLLUP operator

- get another row with summary of the entire set
- applies to columns that aggregate values
- **can not use alias** in ROLLUP operator

```sql
SELECT
	pm.name AS payment_method,
    SUM(p.amount) AS total
FROM payments p
JOIN payment_methods pm
	ON p.payment_method = pm.payment_method_id
GROUP BY pm.name WITH ROLLUP
```

# Writing Complex Queries

## Sub-queries

- A `select` query inside another `select` query

```sql
SELECT *
FROM products
WHERE unit_price > (
	SELECT unit_price
	FROM products
	WHERE product_id = 3
)
```

EX:

```sql
SELECT *
FROM employees
WHERE salary > (
	SELECT 
		AVG(salary)
	FROM employees
    
)
```

### IN subquery

```sql
-- Find the products that have never been ordered before
SELECT *
FROM products
WHERE product_id NOT IN(
	SELECT product_id
    FROM products
    JOIN order_items
    USING (product_id)
);

-- Can also use the following query
SELECT *
FROM products
WHERE product_id NOT IN(
	SELECT DISTINCT product_id
    FROM order_items
);
```

Exercise

```sql
-- Find clients without invoices
SELECT *
FROM clients
WHERE client_id NOT IN (
	SELECT DISTINCT client_id
	FROM invoices
)
```

## Sub-queries VS JOINS

- same result, but different performance and readability

```sql
-- Find clients without invoices
SELECT *
FROM clients
WHERE client_id NOT IN (
	SELECT DISTINCT client_id
	FROM invoices
);

-- USE an inner join
SELECT *
FROM clients
WHERE client_id NOT IN (
	SELECT client_id
	FROM clients
    JOIN invoices
    USING (client_id)
);

-- USE AN OUTER JOIN
SELECT *
FROM clients
LEFT JOIN invoices 
USING (client_id)
WHERE invoice_id IS NULL
```

## ALL keyword

- inter-changeable with `MAX` aggregate function
- where you use the `ALL` keyword, you can use the `MAX` function

```sql
SELECT *
FROM invoices 
WHERE invoice_total > (
	SELECT MAX(invoice_total)
	FROM invoices
	WHERE client_id = 3
)
```

```sql
SELECT *
FROM invoices
WHERE invoice_total > ALL (
	SELECT invoice_total
    FROM invoices
    WHERE client_id = 3
) 
-- invoice_total greater than all results from the sub-query
```

## ANY keyword

```sql
SELECT *
FROM clients
WHERE client_id **IN**(
	SELECT 
		client_id
	FROM invoices
	GROUP BY client_id
	HAVING COUNT(*) >= 2
);

SELECT *
FROM clients
WHERE client_id **= ANY** (
	SELECT 
		client_id
	FROM invoices
	GROUP BY client_id
	HAVING COUNT(*) >= 2
)
```

## Correlated subqueries

- correlation with outer query

```sql
-- select employees whose salary is
-- above the average in their office

-- pseudo-code
-- for each employee
-- 		calculate the avg salary for employee.office
-- 		return the employee if salary > avg
SELECT *
FROM employees e
WHERE salary > (
	SELECT AVG(salary)
	FROM employees
	WHERE office_id = e.office_id
)
```

Exercise

```sql
-- Get invoices that are larger than the
-- client's average invoice amount

SELECT *
FROM invoices i
WHERE invoice_total > (
	SELECT AVG(invoice_total)
    FROM invoices
    WHERE client_id = i.client_id
    )
```

## EXISTS Operator

- check if there is a record that matches the criteria
- more efficient to use the EXISTS operator than IN
    - the sub-query does not return a list

```sql
-- select clients that have an invoice
SELECT *
FROM clients
WHERE client_id IN (
	SELECT DISTINCT client_id
    FROM invoices
);

SELECT *
FROM clientsc
WHERE EXISTs(
	SELECT client_id
    FROM invoices
    WHERE client_id = c.client_id
)
```

Exercise

```sql
-- Find the products that have never been ordered 
SELECT *
FROM products p
WHERE NOT EXISTS (
	SELECT product_id
    FROM order_items
    WHERE product_id = p.product_id
)
```

## Subqueries in the SELECT Clause

```sql
SELECT 
	invoice_id,
    invoice_total,
    (SELECT AVG(invoice_total) 
		FROM invoices) AS invoice_average,
    invoice_total - (SELECT invoice_average) AS difference
FROM invoices
```

Exercise

```sql
SELECT 
	client_id,
    name,
    (SELECT SUM(invoice_total)
		FROM invoices
        WHERE client_id = c.client_id) AS total_sales,
	(SELECT AVG(invoice_total)
		FROM invoices) AS average,
	(SELECT total_sales) - (SELECT average) AS difference
FROM clients c
```

## Subqueries in the FROM Clause

- need to give the subquery in from clause an alias (Whether we will use it or not)
- Use it only for simple queries, or it will become very complicated.
    - (Instead, creating a view to simplify it)

```sql
SELECT *
FROM (
	SELECT 
		client_id,
		name,
		(SELECT SUM(invoice_total)
			FROM invoices
			WHERE client_id = c.client_id) AS total_sales,
		(SELECT AVG(invoice_total)
			FROM invoices) AS average,
		(SELECT total_sales) - (SELECT average) AS difference
	FROM clients c
) AS sales_summary
WHERE total_sales IS NOT NULL
```

# Built-in functions

## Numeric functions

### ROUND()

```sql
SELECT ROUND(5.73, 0) -- keep 0 digit
```

### TRUNCATE()

```sql
SELECT TRUNCATE(5.7342, 2)
```

### CEILING()

```sql
SELECT CEILING(5.7342) -- get the smallest integer greater than or equal to this number
```

### FLOOR()

```sql
SELECT FLOOR(5.7342) -- return the largest integer that is less than or equal to the number
```

### ABS()

- always return a positive value

### RAND()

`SELECT RAND()` 

- generate a random floating number between 0 and 1

## String functions

```sql
SELECT **LENGTH**('sky'); -- 3 characters in sky
SELECT **UPPER**('sky'); -- get the upper case of the string
SELECT **LOWER**('SKY'); -- get the lower case of the string
 
-- remove unnecessory space in strings 
SELECT **LTRIM**('      SKY'); -- left trim
SELECT **RTRIM**('Sky           '); -- right trim
SELECT **TRIM**('      sky       '); -- remove any leading or trailing spaces

-- get the substrings
SELECT **LEFT**('Kindergarten', 4); -- get the left 4 characters in the string
SELECT **RIGHT**('Kindergarten', 6); -- get the right 6 characters in the string
SELECT **SUBSTRING**('Kindergarten', 3, 5); -- get any substring starting from position 3 with length 5

SELECT **LOCATE**('garten', 'Kindergarten'); -- get the location of the position of the first occurance character (not case sensitive)
-- if the searched character isn't exist in the string, would get 0 (instead of -1 in many languages)

SELECT **REPLACE**('Kindergarten', 'garten', 'garden'); -- (original string, sub-string to be replaced, new substring)

SELECT **CONCAT**('first', 'last'); -- concatenating two strings
-- example
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM customers

-- more info refer to google: mysql string functions
```

## Date functions

```sql
SELECT NOW(); -- Date & time
SELECT CURDATE(); -- show the current date without time component
SELECT CURTIME(); -- show the current time without date component

-- extract certain componnets
-- all return integer values
SELECT YEAR(NOW());
SELECT MONTH(NOW());
SELECT HOUR(NOW());
-- return string
SELECT DAYNAME(NOW());
SELECT MONTHNAME(NOW());

SELECT EXTRACT(YEAR FROM NOW())
```

```sql
SELECT *
FROM orders
WHERE YEAR(order_date) < YEAR(NOW())
```

## Formatting

```sql
-- '2019-03-11': this kind of format is not very user friendly

-- Date format function:
-- year
-- %y: a two digits year
-- %Y: a four digits year

-- month
-- %m: two digit month
-- %M: string month

-- day
-- %d

SELECT DATE_FORMAT(NOW(), '%m %y'); 
SELECT DATE_FORMAT(NOW(), '%M %Y'); 

-- Time format function:
-- %H:%i %p  hour:minute am/pm
SELECT TIME_FORMAT(NOW(), '%H:%i %p')
```

## Calculating dates and times

```sql
-- add/ remove a date part to a date-time value
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY); -- return tomorrow date at the current time
SELECT DATE_ADD(NOW(), INTERVAL 1 YEAR); -- return next year at the current time
SELECT DATE_ADD(NOW(), INTERVAL -1 YEAR); -- return last year at the current time
SELECT DATE_SUB(NOW(), INTERVAL 1 YEAR); -- return last year at the current time

-- cal. difference between two date
SELECT DATEDIFF('2019-01-05', '2019-01-01'); -- only return difference in date (not time)
-- cal. difference between two times
SELECT TIME_TO_SEC('9:00'); -- returns number of seconds elapse since midnight 
SELECT TIME_TO_SEC('9:00') - TIME_TO_SEC('9:02')
```

## IFNULL  and COALESCE functions

```sql
**-- IFNULL**
SELECT 
	order_id,
    IFNULL(shipper_id, 'Not assigned') AS shipper -- if the shipper_id is null, show 'Not assigned'
FROM orders;

**-- COALESCE**
-- supply a list of values, the function return the first non null value in the list
SELECT 
	order_id,
    COALESCE(shipper_id, comments, 'Not assigned') AS shipper
FROM orders

```

Exercise

```sql
SELECT 
	CONCAT(first_name, ' ', last_name) AS customer,
    IFNULL(phone, 'Unknown') AS phone
FROM customers
```

## IF function

```sql
-- has three arguments
IF(expression, first, second) -- if the expression is true, return the first value, otherwise, return the second value
```

```sql
SELECT 
	order_id, 
	order_date, 
    IF(
		YEAR(order_date) = YEAR(NOW()), 
        'Active', 
        'Archive') AS status
FROM orders
```

Exercise:

```sql
SELECT 
	p.product_id,
	p.name,
    COUNT(oi.product_id) AS orders,
    IF(
    COUNT(oi.product_id) > 1, 
    'Many times', 
    'Once') 
    AS frequency
FROM products p
JOIN order_items oi 
USING (product_id)
GROUP BY p.product_id, p.name
```

## CASE Operator

- in situations of multiple test expressions

```sql
SELECT 
	order_id,
    CASE
			WHEN YEAR(order_date) = YEAR(NOW()) 
				THEN 'Active'
			WHEN YEAR(order_date) = YEAR(NOW()) - 1
				THEN 'Last year'
			WHEN YEAR(order_date) = YEAR(NOW()) - 2
				THEN 'Archive'
			ELSE 'FUTURE'
		END AS category
FROM orders
```

Exercise

```sql
SELECT
	CONCAT(first_name, ' ', last_name) AS customer,
    points,
    CASE
		WHEN points > 3000 THEN 'Gold'
        WHEN points BETWEEN 2000 and 3000 THEN 'Silver'
        ELSE 'Bronze'
	END AS category
FROM customers
ORDER BY points DESC
```

# Table views

## Creating views

```sql
-- do not create a result, but create a view object
**CREATE VIEW sales_by_client AS**
SELECT 
	 c.client_id,
   c.name,
   SUM(invoice_total) AS total_sales
FROM clients c
JOIN invoices i
USING (client_id)
GROUP BY c.client_id, c.name;

-- SELECT *
-- FROM sales_by_client
-- JOIN clients c
-- USING (client_id)
```

## Altering or Dropping views

```sql
DROP VIEW sales_by_client
```

```sql
**CREATE OR REPLACE VIEW** sales_by_client AS
SELECT 
	c.client_id,
	c.name,
	SUM(invoice_total) AS total_sales
FROM clients c
JOIN invoices i
USING (client_id)
GROUP BY c.client_id, c.name
```

- If cannot access the code, open the “🔧” view, and edit code there.
- the best way is to save the source code in a separate file.

## Updatable Views

```sql
-- if a view does not have any of the following word, it is called a updatable view
-- we can update data through it

-- DISTINCT
-- Aggregrate Functions(MIN, MAX, SUM)
-- GROUP BY / HAVING
-- UNION
```

```sql
CREATE OR REPLACE VIEW invoices_with_balance AS
SELECT 
	invoice_id,
    number,
    client_id,
    invoice_total,
    payment_total,
    invoice_total - payment_total AS balance,
    invoice_date,
    due_date,
    payment_date
FROM invoices
WHERE (invoice_total - payment_total) > 0
```

```sql
DELETE FROM invoices_with_balance
WHERE invoice_id = 1;

UPDATE invoices_with_balance
SET due_date = DATE_ADD(due_date, INTERVAL 2 DAY)
WHERE invoice_id = 2;
```

## With Operation

```sql
UPDATE invoices_with_balance
SET payment_total = invoice_total
WHERE invoice_id = 3;
-- the row with invoice_id = 2 will disappear

CREATE OR REPLACE VIEW invoices_with_balance AS
SELECT 
	invoice_id,
    number,
    client_id,
    invoice_total,
    payment_total,
    invoice_total - payment_total AS balance,
    invoice_date,
    due_date,
    payment_date
FROM invoices
WHERE (invoice_total - payment_total) > 0
WITH CHECK OPTION; -- if a row will disappear, an error will be raised
```

## Other benefits of views

- simplify queries
- reduce the impact of changes
- restrict access to the data

# Stored procedure

## What are stored procedures?

- store and organize SQL
    - a database object contains a block of sql code
    - in the application code, simply code the procedure to add or save data
- Faster execution
- data security
    - limit what users can do using the data

## Creating a stored procedure

```sql
DELIMITER $$
-- change the use of the delimiter (excute all sql blocks in the body part)
-- take all the below statements as one unit 
CREATE PROCEDURE get_clients()
BEGIN
	-- body of the procedure 
	SELECT * FROM clients;
END $$

DELIMITER ; -- change back to the default delimiter 
-- Reminder: do not use any space after the '$$'
```

```sql
CALL get_clients() -- call a procedure in a SQL code
```

Exercise

```sql
DELIMITER $$
CREATE PROCEDURE get_invoices_with_balance()
BEGIN
	SELECT * 
    FROM  invoices_with_balance
    WHERE balance > 0;
END $$
DELIMITER ;
```

## Creating procedures using MySQLWorkbench

- to use MySQL workbench directly can avoid the tedious work of changing the delimiter
    - right click ` stored_procedures` , → create the store_procedure
    - do not need to worry about the usage of delimiter

## Dropping stored procedures

```sql
DROP PROCEDURE get_clients
```

A more safe way to drop: 

```sql
DROP PROCEDURE IF EXISTS get_clients
```

## Parameters

```sql
DROP PROCEDURE IF EXISTS get_clients_by_state;
-- recieve the name of the state
-- return the name of client in that state
DELIMITER $$
CREATE PROCEDURE get_clients_by_state
(
	state CHAR(2) -- a string of two characters
    -- VARCHAR: variable length strings, name, message ... 
)
BEGIN
	SELECT *
    FROM clients c
    WHERE c.state = state;
END $$
DELIMITER ;
```

Exercise

```sql
DROP PROCEDURE IF EXISTS get_invoices_by_client;

DELIMITER $$
CREATE PROCEDURE get_invoices_by_client
(
	client_id INT
)
BEGIN
	SELECT *
    FROM clients c
    WHERE c.client_id = client_id;
END $$
DELIMITER ;
```

### Parameter with default value

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_clients_by_state`(
	state CHAR(2) -- a string of two characters
    -- VARCHAR: variable length strings, name, message ... 
)
BEGIN
	**IF state IS NULL THEN
		SET state = 'CA';
	END IF;** 
	SELECT *
    FROM clients c
    WHERE c.state = state;
END
```

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_clients_by_state`(
	state CHAR(2) -- a string of two characters
    -- VARCHAR: variable length strings, name, message ... 
)
BEGIN
	IF state IS NULL THEN
		SELECT * FROM clients;
	ELSE
		SELECT *
		FROM clients c
		WHERE c.state = state;
	END IF; 
END
```

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_clients_by_state`(
	state CHAR(2) -- a string of two characters
    -- VARCHAR: variable length strings, name, message ... 
)
BEGIN
	IF state IS NULL THEN
		SET state = 'CA';
	END IF; 
	SELECT *
    FROM clients c
    WHERE c.state = state;
END
```

Exercise

```sql
DROP PROCEDURE IF EXISTS get_payments;

DELIMITER $$
CREATE PROCEDURE get_payments
(
	client_id INT(4),
    payment_method_id TINYINT(1)
)
BEGIN 
	SELECT *
    FROM payments p
    JOIN payment_methods pm
    ON p.payment_method = pm.payment_method_id
    WHERE 
		p.client_id = IFNULL(client_id, p.client_id) AND
        p.payment_method = IFNULL(payment_method_id, p.payment_method);
END $$
DELIMITER ;
```

### Parameter validation

- to ensure the procedure does not accidentally store bad data into the database

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `make_payment`(
	invoice_id INT,
    payment_amount DECIMAL(9, 2),
    -- decimal: a number with decimal points (total number of digit, number of digit after the number)
    payment_date DATE
)
BEGIN
	**IF payment_amount <= 0 THEN
		SIGNAL SQLSTATE '22003' 
			SET MESSAGE_TEXT = 'Invalid payment amount'; -- print the error code 
	END IF;**
	UPDATE invoices i
    SET
		i.payment_total = payment_amount,
        i.payment_date = payment_date
	WHERE i.invoice_id = invoice_id;
END
```

### Output parameters

- use parameters to return values to the calling program
- not very convenient, not recommend unless it is needed to use them
    
    ```sql
    CREATE DEFINER=`root`@`localhost` PROCEDURE `get_unpaid_invoices_for_client`(
    	client_id INT,
        OUT invoices_count INT, -- keyword OUT marks them as output parameters
        OUT invoices_total DECIMAL(9, 2)
    )
    BEGIN
    	SELECT COUNT(*), SUM(invoice_total)
        INTO invoices_count, invoices_total -- copy them into the output 
        FROM invoices i
        WHERE i.client_id = client_id 
    		AND payment_total = 0;
    END
    ```
    
    ```sql
    set @invoices_count = 0;
    set @invoices_total = 0; -- define two variables and initialize them to 0
    -- pass the two variables to the procedure
    call sql_invoicing.get_unpaid_invoices_for_client(3, @invoices_count, @invoices_total);
    -- read values from the procedure using select 
    select @invoices_count, @invoices_total;
    ```
    

## Variables

```sql
-- user or session varaible 
SET @invocies_count = 0 -- the variable will be memory during the entire client session
-- when client disconnect mySQL, the variables are freedom

-- local variable: defined inside a procedure or function
-- don't stay in the memory in the entire user session
```

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_risk_factor`()
BEGIN
	-- DECLARE: as soon as they are declared, they are ready for use
	DECLARE risk_factor DECIMAL(9, 2) DEFAULT 0; -- use the declare statement to declare a statement
    DECLARE invoices_total DECIMAL(9, 2); -- set using the SELECT statment, no need to define a default value
    DECLARE invoices_count INT; 
    
    SELECT COUNT(*), SUM(invoice_total)
    INTO invoices_count, invoices_total
    FROM invoices;
    
    SET risk_factor = invoices_total / invoices_count * 5;
    
    SELECT risk_factor;
    -- as soon as the procedure stopped, the decalred variables go out of memory
-- risk_factor = invoices_total / invoices_count * 5
END
```

## Functions

- how to create your own functions?
- good to store the functions into sql file under source control

```sql
CREATE DEFINER=`root`@`localhost` FUNCTION `get_risk_factor_for_client`(
	client_id INT
) RETURNS int
    READS SQL DATA
BEGIN
	DECLARE risk_factor DECIMAL(9, 2) DEFAULT 0; -- use the declare statement to declare a statement
    DECLARE invoices_total DECIMAL(9, 2); -- set using the SELECT statment, no need to define a default value
    DECLARE invoices_count INT; 
    
    SELECT COUNT(*), SUM(invoice_total)
    INTO invoices_count, invoices_total
    FROM invoices i
    WHERE i.client_id = client_id; 
    
    SET risk_factor = invoices_total / invoices_count * 5;
    
	RETURN IFNULL(risk_factor, 0);
END
```

```sql
-- create a function to calcualte the risk factor per client
SELECT 
	client_id,
    name,
    get_risk_factor_for_client(client_id)
FROM clients
```

```sql
DROP FUNCTION IF EXISTS get_risk_factor_for_client;
```

### Other conventions

# Triggers

- a block of SQL code that automatically gets executed before or after an insert, update or delete statement.
- enforce data consistency

```sql
DELIMITER $$

-- the trigger can modify any new table except the table that the triggle is for
CREATE TRIGGER payments_after_insert -- associate with the payment table, fired after insert 
	AFTER INSERT ON payments 
	FOR EACH ROW -- gets fired for each row 
BEGIN
	UPDATE invoices
    SET payment_total = payment_total + NEW.amount -- the NEW keyword indicate the row we just inserted
    WHERE invoice_id = NEW.invoice_id;
END $$

DELIMITER ;
```

```sql
INSERT INTO payments
VALUES (DEFAULT, 5, 3, '2019-01-01', 10, 1)
```

## Viewing trigger

```sql
SHOW TRIGGERS
```

## Dropping trigger

```sql
DROP TRIGGER IF EXISTS payments_after_insert;
```

## Sing Triggers for auditing

- log data to see who made what changes

```sql
-- create a trigger that gets fired when we
-- delete a payment

DROP TRIGGER IF EXISTS payment_after_insert;

DELIMITER $$

CREATE TRIGGER payment_after_insert
	AFTER INSERT ON payments
    FOR EACH ROW
BEGIN
	UPDATE invoices
    SET payment_total = payment_total - NEW.amount
    WHERE invoice_id = NEW.invoice_id;
    
    **INSERT INTO payments_audit
    VALUES (NEW.client_id, NEW.date, NEW.amount, 'Insert', NOW());**
END $$

DELIMITER ; 

DROP TRIGGER IF EXISTS payment_after_delete;

DELIMITER $$

CREATE TRIGGER payment_after_delete
	AFTER DELETE ON payments
    FOR EACH ROW
BEGIN
	UPDATE invoices
    SET payment_total = payment_total - OLD.amount
    WHERE invoice_id = OLD.invoice_id;
    
    **INSERT INTO payments_audit
    VALUES (OLD.client_id, OLD.date, OLD.amount, 'Delete', NOW());**
END $$

DELIMITER ;
```

## Events

- a task (or block of SQL code) that gets executed according to a schedule

```sql
SHOW VARIABLES LIKE 'event%'; -- show variables that only start with 'event'
SET GLOBAL event_scheduler = OFF; -- save system resources
```

```sql
DELIMITER $$
CREATE EVENT yearly_delete_stale_audit_rows
ON SCHEDULE -- how often do you want to proceed the event?
	-- AT '2019-05-01' -- excute only once
    -- EVERY 1 HOUR / 2 DAY/ 1 YEAR 
    EVERY 1 YEAR STARTS '2019-01-01' ENDS '2029-01-01'
DO BEGIN
	DELETE FROM payments_audit
    WHERE action_date < NOW() - INTERVAL 1 YEAR; -- delete all records over 1 year
END $$
DELIMITER ;
```

## Viewing and Dropping or Altering Events

```sql
SHOW EVENTS LIKE 'yearly%';
DROP EVENT IF EXISTS yearly_delete_stale_audit_rows;
ALTER EVENT yearly_delete_stale_audit_rows DISABLE; -- could enable or disable an event
```

# Transactions

- a group of SQL statements that represent a single unit of work
- all statements must be executed correctly or the transaction can not be completed

## Properties

- atomicity: each transaction is a single unit of work no matter how many statements it contains
- consistency: all databases are in consistency
- isolation: each transactions are isolated / protected from each other
- durability: the change made by a transaction is permanent

## How to create a transaction

```sql
USE sql_store;

START TRANSACTION;

INSERT INTO orders (customer_id, order_date, status)
VALUES (1, '2019-01-01', 1);

INSERT INTO order_items 
VALUES (LAST_INSERT_ID(), 1, 1, 1);

COMMIT; 
-- write all the changes into the database
-- if one statement does not successfully execute, all changes would be reverted 

-- ROLLBACK; -- rollback all transactions and redo all statements
```

## Concurrency 并发 and locking

- when executing a second transaction, it has to wait until the first transaction complete
- default lock behaviour

### Concurrency problems

- Lost Updates
    - if does not open the lock, the commitment made by the second transaction will overwrite the first transaction
        - USE LOCK to prevent it from happening
- Dirty Reads
    - a second transaction may read un-committed data from the first transaction (data is dirty)
        - need to provide isolation level for transactions
            - READ COMMITTED — to only read committed data
- Non-repeating Reads
    - Read the same data twice, getting different results
    - isolation level: Repeatable read (even if data is changed by another transaction, we will see the data snapshot with the first read)
- Phantom Reads
    - missing one or more rows in a query
    - isolation level: serializable

![Untitled](SQL%20622e7672942a464b8d92d47cfc81446c/Untitled.png)

### Transaction isolation level

The lower level of isolation can give more concurrency, meaning that more users can access the database at the same time, and more concurrency problems

Higher isolation level, fewer concurrency problems, need more locks and resources.

**Default: Repeatable Read in MySQL**

- Read uncommitted
    - transactions are not isolated or locked from each other (cannot solve any of these problems)
    
    ```sql
    -- user 1
    USE sql_store;
    
    SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
    SELECT points
    FROM customers
    WHERE customer_id = 1; -- will read data that are uncommitted
    ```
    
    ```sql
    -- user 2
    USE sql_store;
    
    START TRANSACTION;
    UPDATE customers
    SET points = 20
    WHERE customer_id = 1;
    COMMIT;
    ```
    
- Read Committed
    - only read transaction results which are committed
    - when two transactions are both committed, may get different results
    
    ```sql
    USE sql_store;
    
    SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
    SELECT points
    FROM customers
    WHERE customer_id = 1; -- will read data that are uncommitted
    ```
    
- Repeatable read
    - guarantee all reads are the same
    
    ```sql
    USE sql_store;
    
    SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
    START TRANSACTION;
    SELECT *
    	FROM customers
    	WHERE state = 'VA'; -- will read data that are uncommitted
    COMMIT;
    ```
    
- Serializable
    - prevents all common problems
    - overhead on the server (memory and CPU)
    - only used for preventing phantom reads
    
    ```sql
    USE sql_store;
    
    SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    START TRANSACTION;
    SELECT *
    	FROM customers
    	WHERE state = 'VA'; -- will read data that are uncommitted
    COMMIT;
    ```
    

To change the isolation level:

```sql
SHOW variables LIKE 'transaction_isolation';
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE; --
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

- deadlock: different transactions can not happen
    
    ```sql
    -- user 1
    USE sql_store;
    
    START TRANSACTION;
    UPDATE customers SET state = 'VA' WHERE customer_id = 1; -- while start update record for customer_id = 1, the record would be locked until commit
    UPDATE orders SET status = 1 WHERE order_id = 1;
    COMMIT;
    ```
    
    ```sql
    -- user 2
    USE sql_store;
    
    START TRANSACTION;
    UPDATE orders SET status = 1 WHERE order_id = 1;
    UPDATE customers SET state = 'VA' WHERE customer_id = 1;
    COMMIT;
    ```
    
    the two transaction will lock each other 
    

# Data Types

- **string type**
    
    Not used in mathematical operations.
    
    `BYTES`: 
    
    English (1 bytes); 
    
    European middle-eastern (2 bytes); 
    
    Asian [Chinese/ Japanese] (3 bytes)
    
    - `CHAR`: fixed length string
    - `VARCHAR`: variable length strings (email, password)
        - `VARCHAR(50)`: for short strings
        - `VARCHAR(255)`: for medium-length strings
    - `MEDIUMTEXT`: stored something longer, 16m characters (json, scv)
    - `LONGTEXT`: max 4GB (textbooks )
    - `TINYTEXT`: max 255 bytes
    - `TEXT`: max 64 KB
- **numeric type**
    - Integer Types
        
        Store whole numbers don’t have decimal points.
        
        Use the smallest data type that suits your need ⇒ smaller size database, faster queries
        
        - `TINYINT` 1b [-128, 127]
        - `UNSIGNED TINYINT` [0, 255]
            - people’s age
        - `SMALLINT` 2b [-32K, 32K]
        - `MEDIUMINT` 3b [-8M, 8M]
        - `INT` 4b [-2B, 2B]
        - `BIGINT` 8b [-9Z, 9Z]
        
        ZEROFILL: use zero to padd
        
        - INT(4) ⇒ 0001
    - Fixed-point and Floating-point types
        - `DECIMAL(p, s)`: fixed point numbers (precision, scale)
            
            p: maximum number of digits [1, 65]
            
            s: number of digits after the decimal points 
            
        - `DEC`
        - `NUMERIC`
        - `FIXED`
        - `FLOAT` 4b
        - `DOUBLE` 8b
    - Booleans types
        - BOOL
        - BOOLEAN
            
            ```sql
            UPDATE posts
            SET is_published = 1 -- or FALSE
            ```
            
    - Enums and Set types
        - change the value in Enums can be expensive
        
        ```sql
        ENUM('SMALL', 'MEDIUM', 'LARGE')
        ```
        
        SET(…)
        
        - also not good to use
- **data and time type**
    - `DATE`
    - `TIME`
    - `DATETIME` 8b
    - `TIMESTAMP` 4b (up to 2038)
    - `YEAR`
- **blob type**
    
    store any binary data
    
    Problems with strong files in a database:
    
    1. increase database size
    2. slower backups
    3. performance problems
    4. more code to read/ write images
    - `TINYBLOB` 255b
    - `BLOB` 65KB
    - `MEDIUMBLOB` 16MB
    - `LONGBLOB` 4GB
- **JSON types**
    - lightweight format for storing and transferring data over the internet
    - store multiple key-value pairs
    
    ```sql
    USE sql_store;
    
    UPDATE products
    SET properties = '
    {
    	"dimension": [1, 2, 3],
        "weight": 10,
        "manufacturer": {"name": "sony"}
    }
    '
    WHERE product_id = 1
    ```
    
    OR
    
    ```sql
    UPDATE products
    SET properties = JSON_OBJECT(
    	'weight', 10,
        'dimensions', JSON_ARRAY(1, 2, 3),
        'manufactor', JSON_OBJECT('name', 'sony')
        )
    WHERE product_id = 1
    ```
    
    Extract from the JSON object:
    
    ```sql
    SELECT product_id, JSON_EXTRACT(properties, '$.weight') AS weight
    FROM products
    WHERE product_id = 1
    ```
    
    ```sql
    SELECT product_id, properties ->> '$.manufactor.name' -- get results with "", can use properties ->> to get results without ""
    FROM products
    WHERE product_id = 1
    ```
    
    Update existed properties or adding new ones
    
    ```sql
    UPDATE products
    SET properties = JSON_SET(
    	properties, -- first arg: the json object we are going to update
        '$.weight', 20,
        '$.age', 10
        )
    WHERE product_id = 1
    ```
    
    Remove properties
    
    ```sql
    UPDATE products
    SET properties = JSON_REMOVE(
    	properties, -- first arg: the json object we are going to update
        '$.age'
        )
    WHERE product_id = 1
    ```
    
- spatial type

# Design well-structured databases

## Data modelling

- process creating a model
1. understand the requirements
    - most important process
    - know the business problem you want to solve
2. build **a conceptual model**
3. build **a logical model** 
    - abstract data model
    - shows the tables columns you need
4. build **a physical model**
    - implementation of the logical model

## Conceptual models

- represents the entities and their relationships
- entity relationship ER & Unified modeling language UML
    - visualization
- ER
    - Microsoft visio
    - draw.io

## Normalization

- **1NF**
    - each cell should have a single value and we cannot have repeated columns
- **2NF**
    - every table should describe one entity, and every column in that table should describe that entity
- **3NF**
    - A column in a table should not be derived from other columns (balance = payment - total)

## Create table

```sql
CREATE DATABASE IF NOT EXISTS sql_store2;
USE sql_store2;
DROP TABLE IF EXISTS customers;
CREATE TABLE IF NOT EXISTS customers
(
	customer_id 	INT PRIMARY KEY AUTO_INCREMENT,
    first_name 		VARCHAR(50) NOT NULL,
    points 			INT NOT NULL DEFAULT 0,
    email 			VARCHAR(50) NOT NULL UNIQUE
);
-- DROP DATABASE IF EXISTS sql_store2;
```

```sql
-- modify existing table
ALTER TABLE customers
	ADD last_name VARCHAR(50) NOT NULL AFTER first_name,
    ADD city	  VARCHAR(50) NOT NULL,
    MODIFY COLUMN first_name VARCHAR(50) DEFAULT '',
    DROP points
    ;
```

## Add relationships

```sql
CREATE DATABASE IF NOT EXISTS sql_store2;
USE sql_store2;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS customers;
CREATE TABLE IF NOT EXISTS customers
(
	customer_id 	INT PRIMARY KEY AUTO_INCREMENT,
    first_name 		VARCHAR(50) NOT NULL,
    points 			INT NOT NULL DEFAULT 0,
    email 			VARCHAR(50) NOT NULL UNIQUE
);
CREATE TABLE orders
(
	order_id 	INT PRIMARY KEY,
    customer_id INT NOT NULL,
    FOREIGN KEY fk_orders_customers (customer_id)
		REFERENCES customers (customer_id)
        ON UPDATE CASCADE
        ON DELETE NO ACTION
);
```

```sql
ALTER TABLE orders
	ADD PRIMARY KEY (order_id),
    DROP PRIMARY KEY,
	DROP FOREIGN KEY fk_orders_customers,
    ADD FOREIGN KEY fk_orders_customers (customer_id)
		REFERENCES customers (customer_id)
        ON UPDATE CASCADE
        ON DELETE NO ACTION;
```

## Character sets and collations

```sql
SHOW CHARSET
```

## Storage engines

Don’t use this in production. 

```sql
ALTER TABLE customers
ENGINE = InnoDB
```