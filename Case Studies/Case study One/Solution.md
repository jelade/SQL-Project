## Case Study #1 - Danny's Diner

````sql
CREATE DATABASE eight_weeks;
create schema eight_weeks;
SET search_path = eight_weeks;
````
### Sales table
````sql
DROP TABLE sales

CREATE TABLE sales 
                ( "customer_id" varchar(1),
					"order_date" DATE,
					"product_id" INTEGER
);

INSERT INTO sales
   ("customer_id", "order_date", "product_id")
   VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
````

### Menu table
````sql

CREATE TABLE menu
 ("product_id"  INTEGER,
 "product_name"  VARCHAR,
 "price" INTEGER);

INSERT INTO menu
("product_id" , "product_name", "price")
VALUES
('1','sushi','10'),
('2', 'curry', '15'),
('3', 'ramen', '12');
````
### Member table
````sql
CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
````  
  
## 1. What is the total amount each customer spent at the restaurant?

### First Join the sales table with the menu table

````sql
SELECT 
	customer_id, 
	sum(price) total_sales
FROM 
	sales s
JOIN 
	menu me ON s.product_id = me.product_id
GROUP BY 
	s.customer_id
ORDER BY 
	customer_id;
````


#### Results:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |



### 2. How many days has each customer visited the restaurant?

````sql
SELECT 
	customer_id, 
	count(distinct order_date) days_visited
FROM 
	sales s
GROUP BY 
	s.customer_id
ORDER BY 
	customer_id;
````


#### Results:

|customer_id| days_visited |
|-----------|--------------|
|A          |             4|
|B          |             6|
|C          |             2|



### 3. What was the first item from the menu purchased by each customer?

````sql
SELECT *
FROM 
	sales s
JOIN 
	menu me ON s.product_id = me.product_id
ORDER BY 
	(customer_id, order_date);

WITH 
	tabb AS (SELECT 
			 customer_id,
			 product_name, 
			 DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) first_item
FROM 
			 sales s 
JOIN 
			 menu me ON s.product_id = me.product_id)

SELECT DISTINCT 
	customer_id, 
	product_name
FROM 
	tabb
WHERE 
	first_item = 1
GROUP BY 
	(customer_id,product_name);
````


#### Results:

|customer_id|product_name|
|-----------|------------|
|A          |curry       |
|A          |sushi       |
|B          |curry       |
|C          |ramen       |


### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT 
	product_name, COUNT(product_name) number_purchased
FROM 
	sales s
JOIN 
	menu me ON s.product_id = me.product_id
GROUP BY
	product_name;
````

#### The most important item on the menu is ramen and it was purchase 8 times



#### Results:

|product_name|number_purchased|
|------------|----------------|
|ramen       |               8|
|sushi       |               3|
|curry       |               4|



### 5. Which item was the most popular for each customer?

````sql
SELECT DISTINCT 
	t1.customer_id, 
	t1.product_name, 
	t1.purchased_count
from (SELECT 
	  customer_id, 
	  product_name, 
	  COUNT(product_name) purchased_count, 
	  DENSE_RANK() over(partition by customer_id order by COUNT(s.customer_id) DESC ) as most_popular
	FROM 
	  sales s
	JOIN 
	  menu me ON s.product_id = me.product_id
	GROUP BY 
	  (customer_id,product_name)) AS t1
	WHERE 
		most_popular = 1;
````

#### Results:

|customer_id|product_name|purchased_count |
|-----------|------------|----------------|
|A          |ramen       |               3|
|B          |curry       |               2|
|B          |ramen       |               2|
|B          |sushi       |               2|
|C          |ramen       |               3|


### 6. Which item was purchased first by the customer after they became a member?

````sql
SELECT *
FROM 
	sales s
JOIN 
	menu me ON s.product_id = me.product_id
FULL JOIN 
	members mem ON s.customer_id = mem.customer_id
ORDER BY 
	s.customer_id;


SELECT DISTINCT 
	s.customer_id,
	mem.join_date,
	first_value(product_name) over(partition by s.customer_id order by s.order_date ASC) as first_purchase
FROM 
	sales s
JOIN 
	menu me ON s.product_id = me.product_id
FULL JOIN 
	members mem ON s.customer_id = mem.customer_id
WHERE 
	s.order_date >= mem.join_date;
````


#### Results:

|customer_id|join_date |first_purchase|
|-----------|----------|--------------|
|B          |2021-01-09|sushi         |
|A          |2021-01-07|curry         |


*/

--7. Which item was purchased just before the customer became a member?

````sql

SELECT DISTINCT 
	s.customer_id,mem.join_date,
	first_value(product_name) over(partition by s.customer_id order by s.order_date DESC) as first_purchase
FROM 
	sales s
JOIN 
	menu me ON s.product_id = me.product_id
FULL JOIN 
	members mem ON s.customer_id = mem.customer_id
WHERE 
	s.order_date < mem.join_date;
````


#### Results:

|customer_id|join_date |first_purchase|
|-----------|----------|--------------|
|B          |2021-01-09|sushi         |
|A          |2021-01-07|sushi         |



### 8. What is the total items and amount spent for each member before they became a member?

````sql
SELECT 
	s.customer_id, 
	sum(me.price) total_spent
FROM 
	sales s
JOIN 
	menu me ON s.product_id = me.product_id
FULL JOIN 
	members mem ON s.customer_id = mem.customer_id
WHERE 
	s.order_date < mem.join_date
GROUP BY 
	(s.customer_id);
````


#### Results:

|customer_id|total_spent|
|-----------|-----------|
|B          |         40|
|A          |         25|



### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

````sql
SELECT 
	customer_id, 
	sum(customer_points) AS customer_total_points
FROM( SELECT 
	 s.customer_id, 
	 me.price, 
	 me.product_name,
(CASE 
 	WHEN me.product_name = 'sushi' then price*20
 	ELSE price*10
	END) AS customer_points
FROM 
	 sales s
JOIN 
	 menu me ON s.product_id = me.product_id
	ORDER BY customer_id) as tot
GROUP BY customer_id;
````



#### Results:

|customer_id|customer_total_points|
|-----------|---------------------|
|A          |                  860|
|B          |                  940|
|C          |                  360|



### In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

````sql
WITH sum_tab AS (
    SELECT 
        s.customer_id, 
        me.price, 
        me.product_name, 
        s.order_date, 
        mem.join_date,
        (CASE 
            WHEN s.order_date BETWEEN mem.join_date AND mem.join_date + INTERVAL '7' DAY THEN me.price * 20
            ELSE me.price * 10
         END) AS customer_points
    FROM 
        sales s
    JOIN 
        menu me ON s.product_id = me.product_id
    JOIN 
        members mem ON s.customer_id = mem.customer_id
)
SELECT 
    customer_id, 
    SUM(customer_points) AS customer_total_points
FROM 
    sum_tab
WHERE 
    order_date < TO_DATE('01-FEB-2021', 'DD-MON-YYYY')
GROUP BY 
    customer_id;
````


#### Results:

|customer_id|customer_total_points|
|-----------|---------------------|
|A          |                 1270|
|B          |                  820|


### Create a new column which will be membership status for each purchase

````sql
SELECT *
FROM 
	sales s
JOIN 
	menu me ON s.product_id = me.product_id
FULL JOIN
	members mem ON s.customer_id = mem.customer_id;


SELECT 
	tabb.customer_id, 
	tabb.order_date, 
	tabb.product_name, 
	tabb.price, 
CASE 
 	WHEN tabb.order_date >= tabb.join_date THEN 'Y' 
 	ELSE 'N' END as member
FROM 
	(SELECT 
	 	s.customer_id, 
	 	me.price, 
	 	me.product_name, 
	 	s.order_date, 
	 	mem.join_date
FROM 
	 sales s
JOIN 
	 menu me ON s.product_id = me.product_id
FULL JOIN 
	 members mem ON s.customer_id = mem.customer_id) as tabb
ORDER BY 
	(tabb.customer_id, tabb.order_date) asc;
````

### Rank date of purchase when the customer became a member

````sql
WITH table1 AS (
    SELECT 
        tabb.customer_id, 
        tabb.order_date, 
        tabb.product_name, 
        tabb.price, 
        CASE 
            WHEN tabb.order_date >= tabb.join_date THEN 'Y' 
            ELSE 'N' 
        END AS member_
    FROM (
        SELECT 
            s.customer_id, 
            me.price, 
            me.product_name, 
            s.order_date, 
            mem.join_date
        FROM sales s
        JOIN menu me ON s.product_id = me.product_id
        FULL JOIN members mem ON s.customer_id = mem.customer_id
    ) AS tabb
)

SELECT 
	tabb1.customer_id, 
	tabb1.order_date, 
	tabb1.product_name, 
	tabb1.price, tabb1.member_, 
CASE 
	WHEN tabb1.member_ = 'Y' THEN  RANK() OVER(PARTITION BY tabb1.customer_id, member_ ORDER BY tabb1.order_date) 
	END AS ranking
FROM 
	table1 tabb1;
````


### Results:

|customer_id|order_date|product_name|price |member|ranking|
|-----------|----------|------------|------|------|-------|
|A          |2021-01-01|sushi       |    10|N     | [null]|
|A          |2021-01-01|curry       |    15|N     | [null]|
|A          |2021-01-07|curry       |    15|Y     |      1|
|A          |2021-01-10|ramen       |    12|Y     |      2|
|A          |2021-01-11|ramen       |    12|Y     |      3|
|A          |2021-01-11|ramen       |    12|Y     |      3|
|B          |2021-01-01|curry       |    15|N     | [null]|
|B          |2021-01-02|curry       |    15|N     | [null]|
|B          |2021-01-04|sushi       |    10|N     | [null]|
|B          |2021-01-11|sushi       |    10|Y     |      1|
|B          |2021-01-16|ramen       |    12|Y     |      2|
|B          |2021-02-01|ramen       |    12|Y     |      3|
|C          |2021-01-01|ramen       |    12|N     | [null]|
|C          |2021-01-01|ramen       |    12|N     | [null]|
|C          |2021-01-07|ramen       |    12|N     | [null]|

