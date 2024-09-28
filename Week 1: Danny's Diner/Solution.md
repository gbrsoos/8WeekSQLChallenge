DESCRIPTION HERE ABOUT THIS WEEK'S SETUP

## Step 1: Getting hold of the database from [DB-Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

```sql
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
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
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```

## Question 1: What is the total amount each customer spent at the restaurant?
```sql
SELECT sales.customer_id, 
	SUM(menu.price) AS total_spent
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY total_spent ASC;
```

|  | customer_id  | total_spent   |
|--|--------------|---------------|
| 1| C            | 36            |
| 2| B            | 74            |
| 3| A            | 76            |

## Question 2: How many days has each customer visited the restaurant?
```sql
SELECT customer_id, 
	COUNT(DISTINCT order_date) 
FROM dannys_diner.sales
GROUP BY customer_id;
```

|  | customer_id  | count  	  |
|--|--------------|---------------|
| 1| A            | 4             |
| 2| B            | 6             |
| 3| C            | 2             |

## Question 3: What was the first item from the menu purchased by each customer?
```sql
WITH merged_purchases AS (
SELECT sales.customer_id,
	sales.order_date,
	menu.product_name,
	DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS row_num
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
)

SELECT customer_id,
	product_name
FROM merged_purchases
WHERE row_num = 1
GROUP BY customer_id, product_name;
```

|  | customer_id  | count  	  |
|--|--------------|---------------|
| 1| A            | curry         |
| 2| A            | sushi         |
| 3| B            | curry         |
| 4| C		  | ramen         |

## Question 4: What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT product_name, COUNT(product_name) AS times_purchased FROM dannys_diner.menu
INNER JOIN dannys_diner.sales
	ON menu.product_id = sales.product_id
GROUP BY product_name
ORDER BY times_purchased DESC
LIMIT 1;
```

|  | product_name | purchased  	  |
|--|--------------|---------------|
| 1| ramen        | 8             |

## Question 5: Which item was the most popular for each customer?
```sql
WITH most_popular AS (
	SELECT sales.customer_id,
			menu.product_name,
			COUNT(menu.product_id),
			DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(sales.customer_id) DESC) as purchase_rank
	FROM dannys_diner.sales
	INNER JOIN dannys_diner.menu
		ON sales.product_id = menu.product_id
	GROUP BY sales.customer_id, menu.product_name
)

SELECT customer_id,
		product_name,
		count
FROM most_popular
WHERE purchase_rank = 1
ORDER BY customer_id
```

|  | customer_id  | product_name  | count  |
|--|--------------|---------------|--------|
| 1| A            | ramen         | 3      |
| 2| B            | sushi         | 2      | 
| 3| B            | curry         | 2      |
| 4| B		  | ramen         | 2      |
| 5| C		  | ramen         | 3      |

## Question 6: Which item was purchased first by the customer after they became a member?
```sql
WITH join_date AS(
	SELECT
		members.customer_id,
		sales.product_id,
		ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date) AS row_number
	FROM dannys_diner.members
	INNER JOIN dannys_diner.sales
		ON members.customer_id = sales.customer_id
		AND sales.order_date > members.join_date
)

SELECT 
	customer_id,
	menu.product_name
FROM join_date
INNER JOIN dannys_diner.menu
	ON join_date.product_id = menu.product_id
WHERE row_number = 1
ORDER BY customer_id ASC;
```

|  | customer_id  | product_name  |
|--|--------------|---------------|
| 1| A	          | ramen         |
| 1| B	          | sushi         |

## Question 7: Which item was purchased just before the customer became a member?
```sql
WITH premember_purchases AS (
	SELECT 
		members.customer_id,
		sales.product_id,
	ROW_NUMBER() OVER(PARTITION BY members.customer_id ORDER BY sales.order_date DESC) as ranking
	FROM dannys_diner.members
	INNER JOIN dannys_diner.sales
		ON members.customer_id = sales.customer_id
	AND sales.order_date < members.join_date
)

SELECT
	customer_id,
	menu.product_name
FROM premember_purchases
INNER JOIN dannys_diner.menu
	ON premember_purchases.product_id = menu.product_id
WHERE ranking = 1
ORDER BY customer_id ASC;
```

|  | customer_id  | product_name  |
|--|--------------|---------------|
| 1| A	          | sushi         |
| 1| B	          | sushi         |

## Question 8: What is the total items and amount spent for each member before they became a member?
```sql
SELECT
	sales.customer_id,
	COUNT(sales.product_id) AS num_products,
	SUM(menu.price) AS sum_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members
	ON sales.customer_id = members.customer_id
	AND sales.order_date < members.join_date
GROUP BY sales.customer_id
ORDER BY customer_id ASC;
```

|  | customer_id  | num_products  |amount_spent |
|--|--------------|---------------|-------------|
| 1| A	          | 2             | 25  	|
| 1| B	          | 3             | 40  	|




