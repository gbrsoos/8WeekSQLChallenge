## Week 1 - Danny's Diner
![8WeekSQLChallenge - Week2](https://8weeksqlchallenge.com/images/case-study-designs/2.png)

### Introduction
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

### Database
#### Entity Relationship Diagram
![Week 2 - Entity Relationship Diagram](utils/PizzaRunner_dbdiagram.png)

As visualized on the diagram and written above, the database consists of 3 tables: sales, members, and menu. The primary connecting variables are customer_id and product_id, these are going to play a crucial role in conducting the queries.

## Solution
### Step 1: Getting hold of the database from [DB-Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
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

### Question 1: What is the total amount each customer spent at the restaurant?
To answer this question, `customer_id` and `price` will both be needed. To tackle this, I used `INNER_JOIN` (but casual `JOIN` also does the trick in this case) on the `product_id` column to merge `dannys_diner.sales` and `dannys_diner.menu`, since the two columns of need are present in these two tables. After this is done, the `SUM()` of the prices was calculated to get the total spending, and the output was grouped by the `customer_id`.

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

### Question 2: How many days has each customer visited the restaurant?
This question can be answered in a simple way. No `JOIN` is needed, since all the required information is present in the `dannys_diner.sales` table. All I had to do is to count the order dates for each `customer_id`, and then use `GROUP BY`.

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

### Question 3: What was the first item from the menu purchased by each customer?
This is the first question where a Casual Table Expression (CTE) is required. We need to create a temporary table including the `customer_id`, the `sales_date`, the `product_name`, and an additional column that calculates the 'rank' of each item purchased by each customer. To create this column, `DENSE_RANK()` was used, which gives a unique number for the unique values (starting from 1) without leaving a gap in the numbering. `PARTITION BY` is responsible for separating the ranking between each `customer_id`, so the ranking is separate, and I use `ORDER BY sales.order_date` to take the order with the earliest date as 1. After this, I create the output by selecting `customer_id`, `product_name` from the CTE and limit it to only show instances where the rank is 1. The reason why A is present in the output two times is because they ordered both curry and sushi in their first order.

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

### Question 4: What is the most purchased item on the menu and how many times was it purchased by all customers?
To have this counted, I first used `INNER JOIN` to have the `sales` and the `menu` table joined, and performed `COUNT` to see how many times each product occurs. After the purchase numbers have been obtained, the rows have been sorted in descending order based on the occurence (`times_purchased`), a displayed only the top row with `LIMIT 1`.

```sql
SELECT product_name, COUNT(product_name) AS times_purchased FROM dannys_diner.menu
INNER JOIN dannys_diner.sales
	ON menu.product_id = sales.product_id
GROUP BY product_name
ORDER BY times_purchased DESC
LIMIT 1;
```

|  | product_name | times_purchased  	  |
|--|--------------|---------------|
| 1| ramen        | 8             |

### Question 5: Which item was the most popular for each customer?
To see the most popular items for each customer, a CTE is used once again. After using `INNER JOIN` on the `menu` and `sales` tables, I gathered the `customer_id`, the `product_name` and the number of times each product has been purchased by each customer (`COUNT(menu.product_id)`). Then obtained the `DENSE_RANK()` of each product based on how many times they were bought. Thus, in the outer query I only had to filter this `rank` to one to see the most liked products for each customer. 

```sql
WITH most_popular AS (
	SELECT sales.customer_id,
			menu.product_name,
			COUNT(menu.product_id),
			DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(menu.product_name) DESC) as purchase_rank
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

### Question 6: Which item was purchased first by the customer after they became a member?
In the `join_date` CTE, after joining `dannys_diner.sales` and `dannys_diner.members`, I used ROW_NUMBER() to give a unique `row_number` to each purchase by each `customer_id` and filtered the purchases to only query where the `order_date` is bigger than the `join_date`. In the outer query afterwards, I only filtered to show the cases where the `row_number = 1`, so the first purchase after becoming a member. C is not present here, since they never became a member.

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
| 2| B	          | sushi         |

### Question 7: Which item was purchased just before the customer became a member?
This solution is pretty similar to Question 6, but with a little twist. Here in the `ROW_NUMBER()` I sorted the `order_date` in descending order to capture the last purchase instead of the first. After this I used `sales.order_date < member.join_date` to capture all purchases that happenned before becoming a member. The outer query is responsible for the same tasks as in Question 6.

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
| 2| B	          | sushi         |

### Question 8: What is the total items and amount spent for each member before they became a member?
For this a double `INNER JOIN` is required, since data is needed from all three tables. Additionally, when joining `dannys_diner.members` and `dannys_diner.sales`, I filtered the purchases by `sales.order_date < members.join_date` to only see purchases before membership. To see the number of items purchased, I counted the product_ids, and used `SUM(menu.price)` to have the aggregated expense. A bought 2 items before becoming a member with a total price of 25, and B purchased 3 items that are worth 40.

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
| 2| B	          | 3             | 40  	|

### Question 9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?
This is the first question where `CASE` needs to be incorporated. Since the `product_id` of `sushi` is `1`, the number of points given needs to be doubled if `product_id = 1`. The points are stored in a CTE, and the outer query is responsible for aggregating the points for each `customer_id`.

```sql
WITH points_CTE AS(
	SELECT 
		menu.product_id,
		CASE
			WHEN product_id = 1 THEN price * 20
			ELSE price * 10 END AS points
	FROM dannys_diner.menu
)

SELECT
	sales.customer_id,
	SUM(points_CTE.points)
FROM dannys_diner.sales
INNER JOIN points_CTE
	ON sales.product_id = points_CTE.product_id
GROUP BY customer_id
ORDER BY customer_id ASC;
```

|  | customer_id  | sum_points    |
|--|--------------|---------------|
| 1| A	          | 860           |
| 2| B	          | 940           |
| 3| C	          | 360           |

### Question 10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?
Here, I had to play with the dates as well. When creating the CTE `dates_CTE`, I created a new column `closing_day`, which is a date 6 days later to the `join_date` (this is because the program lasts for one week INCLUDING THE JOIN DATE). I created another column, which represents the end of January, since that is the end of the calculation window. To do so, I used a generalized method which is not limited down to this special usecase. Here I use `DATE_TRUNC('month', '2021-01-31'::DATE)` which truncates the given date back to the first day of the month, the with `+ interval '1 month'` I add an extra month and with `- interval '1 day'` I subtract a day which ends in the last day of January. With this approach, the calculation can be done for all dates dynamically.
In the outer query, `CASE` is used again to double the points for `sushi`, and to double the points for each purchase if the order happenned between the `join_date` and the last day of the program `closing_day`.

```sql
WITH dates_CTE AS (
	SELECT 
		members.customer_id,
		join_date,
		join_date + 6 AS closing_day,
		DATE_TRUNC(
			'month', '2021-01-31'::DATE)
			+ interval '1 month'
			- interval '1 day' AS end_of_jan
	FROM dannys_diner.members
)

SELECT 
	sales.customer_id,
	SUM(CASE
			WHEN menu.product_id = 1 THEN menu.price * 20
			WHEN sales.order_date BETWEEN dates_CTE.join_date AND dates_CTE.closing_day THEN menu.price * 20
			ELSE menu.price * 10 END) AS total_points
FROM dannys_diner.sales
INNER JOIN dates_CTE
	ON sales.customer_id = dates_CTE.customer_id
	AND dates_CTE.join_date <= sales.order_date
	AND sales.order_date <= dates_CTE.end_of_jan
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY customer_id ASC;
```

|  | customer_id  | total_points  |
|--|--------------|---------------|
| 1| A	          | 1020          |
| 2| B	          | 320           |

### Bonus 1: Join All The Things
The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

Recreate the following table output using the available data:
| customer_id   | order_date   | product_name   |   price | member   |
|:--------------|:-------------|:---------------|--------:|:---------|
| A             | 2021-01-01   | curry          |      15 | N        |
| A             | 2021-01-01   | sushi          |      10 | N        |
| A             | 2021-01-07   | curry          |      15 | Y        |
| A             | 2021-01-10   | ramen          |      12 | Y        |
| A             | 2021-01-11   | ramen          |      12 | Y        |
| A             | 2021-01-11   | ramen          |      12 | Y        |
| B             | 2021-01-01   | curry          |      15 | N        |
| B             | 2021-01-02   | curry          |      15 | N        |
| B             | 2021-01-04   | sushi          |      10 | N        |
| B             | 2021-01-11   | sushi          |      10 | Y        |
| B             | 2021-01-16   | ramen          |      12 | Y        |
| B             | 2021-02-01   | ramen          |      12 | Y        |
| C             | 2021-01-01   | ramen          |      12 | N        |
| C             | 2021-01-01   | ramen          |      12 | N        |
| C             | 2021-01-07   | ramen          |      12 | N        |

```sql
SELECT 
	sales.customer_id,
	sales.order_date,
	menu.product_name,
	menu.price,
	CASE
		WHEN members.join_date > sales.order_date THEN 'N'
		WHEN members.join_date <= sales.order_date THEN 'Y'
		ELSE 'N' END AS membership
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
LEFT JOIN dannys_diner.members
	ON sales.customer_id = members.customer_id
ORDER BY customer_id, order_date;
```

| customer_id   | order_date   | product_name   |   price | member   |
|:--------------|:-------------|:---------------|--------:|:---------|
| A             | 2021-01-01   | curry          |      15 | N        |
| A             | 2021-01-01   | sushi          |      10 | N        |
| A             | 2021-01-07   | curry          |      15 | Y        |
| A             | 2021-01-10   | ramen          |      12 | Y        |
| A             | 2021-01-11   | ramen          |      12 | Y        |
| A             | 2021-01-11   | ramen          |      12 | Y        |
| B             | 2021-01-01   | curry          |      15 | N        |
| B             | 2021-01-02   | curry          |      15 | N        |
| B             | 2021-01-04   | sushi          |      10 | N        |
| B             | 2021-01-11   | sushi          |      10 | Y        |
| B             | 2021-01-16   | ramen          |      12 | Y        |
| B             | 2021-02-01   | ramen          |      12 | Y        |
| C             | 2021-01-01   | ramen          |      12 | N        |
| C             | 2021-01-01   | ramen          |      12 | N        |
| C             | 2021-01-07   | ramen          |      12 | N        |

### Bonus 2: Rank All The Things
Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program

```sql
WITH base_CTE AS(
	SELECT 
		sales.customer_id,
		sales.order_date,
		menu.product_name,
		menu.price,
		CASE
			WHEN members.join_date > sales.order_date THEN 'N'
			WHEN members.join_date <= sales.order_date THEN 'Y'
			ELSE 'N' END AS membership
	FROM dannys_diner.sales
	INNER JOIN dannys_diner.menu
		ON sales.product_id = menu.product_id
	LEFT JOIN dannys_diner.members
		ON sales.customer_id = members.customer_id
	ORDER BY customer_id, order_date
)

SELECT 
	*,
	CASE
		WHEN membership = 'N' THEN NULL
		ELSE DENSE_RANK() OVER (PARTITION BY customer_id, membership ORDER BY order_date) END AS ranking
FROM base_CTE;
```

| customer_id   | order_date   | product_name   |   price | member   |   rank |
|:--------------|:-------------|:---------------|--------:|:---------|-------:|
| A             | 2021-01-01   | sushi          |      10 | N        |   null |
| A             | 2021-01-01   | curry          |      15 | N        |   null |
| A             | 2021-01-07   | curry          |      15 | Y        |      1 |
| A             | 2021-01-10   | ramen          |      12 | Y        |      2 |
| A             | 2021-01-11   | ramen          |      12 | Y        |      3 |
| A             | 2021-01-11   | ramen          |      12 | Y        |      3 |
| B             | 2021-01-01   | curry          |      15 | N        |   null |
| B             | 2021-01-02   | curry          |      15 | N        |   null |
| B             | 2021-01-04   | sushi          |      10 | N        |   null |
| B             | 2021-01-11   | sushi          |      10 | Y        |      1 |
| B             | 2021-01-16   | ramen          |      12 | Y        |      2 |
| B             | 2021-02-01   | ramen          |      12 | Y        |      3 |
| C             | 2021-01-01   | ramen          |      12 | N        |   null |
| C             | 2021-01-01   | ramen          |      12 | N        |   null |
| C             | 2021-01-07   | ramen          |      12 | N        |   null |



