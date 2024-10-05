## Week 2/D - Pricing and Ratings
![8WeekSQLChallenge - Week2](https://8weeksqlchallenge.com/images/case-study-designs/2.png)

### Introduction
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

### Database
#### Entity Relationship Diagram
![Week 2 - Entity Relationship Diagram](utils/PizzaRunner_dbdiagram.png)

As visualized on the diagram and written above, the database consists of 6 tables: runner_orders, runners, customer_orders, pizza_names, pizza_recipes, and pizza_toppings. The primary connecting variables are customer_id, order_id, runner_id, pizza_id, and topping_id. These are going to play a crucial role in conducting the queries.

## Solution
### Step 1: Getting hold of the database from [DB-Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

<details>
  <summary>Click to expand SQL script</summary>

```sql
CREATE SCHEMA pizza_runner;
SET search_path = pizza_runner;

DROP TABLE IF EXISTS runners;
CREATE TABLE runners (
  "runner_id" INTEGER,
  "registration_date" DATE
);
INSERT INTO runners
  ("runner_id", "registration_date")
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');


DROP TABLE IF EXISTS customer_orders;
CREATE TABLE customer_orders (
  "order_id" INTEGER,
  "customer_id" INTEGER,
  "pizza_id" INTEGER,
  "exclusions" VARCHAR(4),
  "extras" VARCHAR(4),
  "order_time" TIMESTAMP
);

INSERT INTO customer_orders
  ("order_id", "customer_id", "pizza_id", "exclusions", "extras", "order_time")
VALUES
  ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
  ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
  ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
  ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
  ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
  ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
  ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
  ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
  ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
  ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
  ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');


DROP TABLE IF EXISTS runner_orders;
CREATE TABLE runner_orders (
  "order_id" INTEGER,
  "runner_id" INTEGER,
  "pickup_time" VARCHAR(19),
  "distance" VARCHAR(7),
  "duration" VARCHAR(10),
  "cancellation" VARCHAR(23)
);

INSERT INTO runner_orders
  ("order_id", "runner_id", "pickup_time", "distance", "duration", "cancellation")
VALUES
  ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
  ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
  ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
  ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
  ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
  ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
  ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
  ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
  ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
  ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');


DROP TABLE IF EXISTS pizza_names;
CREATE TABLE pizza_names (
  "pizza_id" INTEGER,
  "pizza_name" TEXT
);
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');


DROP TABLE IF EXISTS pizza_recipes;
CREATE TABLE pizza_recipes (
  "pizza_id" INTEGER,
  "toppings" TEXT
);
INSERT INTO pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
  (2, '4, 6, 7, 9, 11, 12');


DROP TABLE IF EXISTS pizza_toppings;
CREATE TABLE pizza_toppings (
  "topping_id" INTEGER,
  "topping_name" TEXT
);
INSERT INTO pizza_toppings
  ("topping_id", "topping_name")
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');
```
</details>

<details>
    <summary>There is a part of data cleaning and datatype conversion, which was already discussed in Week 2/A. so I just collapsed it. Feel free to inspect tho.</summary>
  
Before diving into the questions, Danny gave some hints about 2 tables, `customer_orders` and `runner_orders`. After a quick query, it became clear that there are multiple columns that are not in the correct data type and there are `NULL` values that are stored as strings. First, let us tackle these issues.

### NULL values
```sql
UPDATE pizza_runner.customer_orders
SET exclusions = NULL
WHERE exclusions = 'null';

UPDATE pizza_runner.customer_orders
SET extras = NULL
WHERE extras = 'null';

UPDATE pizza_runner.customer_orders
SET exclusions = NULL
WHERE exclusions = '';

UPDATE pizza_runner.customer_orders
SET extras = NULL
WHERE extras = '';

UPDATE pizza_runner.runner_orders
SET pickup_time = NULL
WHERE pickup_time = 'null';

UPDATE pizza_runner.runner_orders
SET distance= NULL
WHERE distance = 'null';

UPDATE pizza_runner.runner_orders
SET duration = NULL
WHERE duration = 'null';

UPDATE pizza_runner.runner_orders
SET cancellation = NULL
WHERE cancellation = 'null';

UPDATE pizza_runner.runner_orders
SET cancellation = NULL
WHERE cancellation = '';
```

### Data types
```sql
ALTER TABLE pizza_runner.customer_orders
ALTER COLUMN exclusions TYPE INTEGER;

UPDATE pizza_runner.customer_orders
SET exclusions = string_to_array(exclusions, ', ')::INTEGER[]
WHERE exclusions IS NOT NULL;

ALTER TABLE pizza_runner.customer_orders
ALTER COLUMN extras TYPE TEXT;

UPDATE pizza_runner.customer_orders
SET extras = string_to_array(extras, ', ')::INTEGER[]
WHERE extras IS NOT NULL;

UPDATE pizza_runner.runner_orders
SET distance = TRIM(REGEXP_REPLACE(distance, '[^0-9.]+', ''))
WHERE distance IS NOT NULL;

ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN distance TYPE FLOAT
USING distance::FLOAT

UPDATE pizza_runner.runner_orders
SET duration = TRIM(REGEXP_REPLACE(duration, '[^0-9]+', ''))
WHERE duration IS NOT NULL;

ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN duration TYPE FLOAT
USING duration::FLOAT
```
After running these queries, all columns have been cleaned, so the solving process can be started.
</details>

### Task 0: Altering `search_path`
I am going to be blunt here, I got bored writing pizza_runner all over and over again, so I just alter the `search_path` for the actual session, which will spare me a lot of time and unnecessary effort. So even though I am not going to refer to pizza_runner again, I am going to continue using the same schema as I was thus far.

```sql
SET search_path TO pizza_runner;
```

### Question 1: If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
After adding the extra column that amtches the given prices to the pizzas, I just used `SUM()` to see the overall price, which is $160.

```sql
SELECT
	'$' || SUM(
		CASE
			WHEN co.pizza_id = 1 THEN 12
			ELSE 10 END) AS total_income
FROM runner_orders AS ro
JOIN customer_orders AS co
	ON ro.order_id = co.order_id;
```

|  | total_income |
|--|--------------|
| 1| $160          |


### Question 2: What if there was an additional $1 charge for any pizza extras? `Add cheese is $1 extra`
Herre I had to circumvent the presence of arrays. To do so, I exploited `regexp_replace()`, which enabled me to remove `{` and `}` from every array, and then convert each number in the `customer_orders.extras` column to see if there was extra Cheese added. If so, I add +$1 to the extra price. Also, since I had two `CASE()` functions applied, I broke the query up to a CTE and an outer query. The results say that there was only one occasion where Cheese was added as an extra.

```sql
WITH income_per_pizza_CTE AS(
	SELECT
		ro.*,
		co.pizza_id,
		co.extras,
		CASE
			WHEN co.pizza_id = 1 THEN 12
			ELSE 10 END AS total_income_base
	FROM runner_orders AS ro
	JOIN customer_orders AS co
		ON ro.order_id = co.order_id
)

SELECT
	'$' || SUM(CASE
                WHEN 4 = ANY(string_to_array(regexp_replace(extras, '[{}]', '', 'g'), ',')::INT[]) 
					THEN total_income_base + 1
				ELSE total_income_base END) AS total_income
FROM income_per_pizza_CTE
```

|  | total_income |
|--|--------------|
| 1| $160          |

### Question 3: The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
Here I displayed all the queries I ran. In the first step, I created the new schema. In the second one, I created the base `ratings` table, which includes everything from the `runner_orders` table, and the `customer_id` from the `custoemr_orders` table, to have all the needed information. `DISTINCT` is required here to only keep one row for each `order_id`. After this, I created the `ratings` column, and I assigned a good-hearted 5 to each order which was successful, so if `cancellation IS NULL`.

```sql
CREATE SCHEMA pizza_runner_ratings;
```
```sql
CREATE TABLE pizza_runner_ratings.ratings AS
SELECT	DISTINCT
	co.customer_id,
	ro.*
FROM pizza_runner.customer_orders AS co
LEFT JOIN pizza_runner.runner_orders AS ro
	ON co.order_id = ro.order_id;
```
```sql
ALTER TABLE pizza_runner_ratings.ratings
ADD COLUMN ratings INTEGER;
```
```sql
UPDATE pizza_runner_ratings.ratings
SET ratings = '5'
WHERE cancellation IS NULL;
```

| customer_id | order_id | runner_id | pickup_time         | distance | duration | cancellation            | ratings |
|-------------|----------|-----------|---------------------|----------|----------|--------------------------|---------|
| 101         | 6        | 3         |                     |          |          | Restaurant Cancellation  |         |
| 103         | 9        | 2         |                     |          |          | Customer Cancellation    |         |
| 105         | 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                          | 5       |
| 102         | 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                          | 5       |
| 102         | 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                          | 5       |
| 104         | 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                          | 5       |
| 101         | 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                          | 5       |
| 103         | 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                          | 5       |
| 104         | 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                          | 5       |
| 101         | 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                          | 5       |

### Question 4: Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
`customer_id`
`order_id`
`runner_id`
`rating`
`order_time`
`pickup_time`
`Time between order and pickup`
`Delivery duration`
`Average speed`
`Total number of pizzas`
Since the last question wanted to aggregate to orders rather than elements of orders, I had to create a CTE where I count the number of pizzas per order. Additionally, I did not include `order_time`, so I needed to attach that as well using `JOIN`. 

```sql
WITH pizza_count_CTE AS (
	SELECT
		order_id,
		COUNT(order_id) AS pizza_num
	FROM pizza_runner.customer_orders
	GROUP BY order_id
)

SELECT DISTINCT
	r.customer_id,
	r.order_id,
	r.runner_id,
	r.ratings,
	co.order_time,
	r.pickup_time,
	ROUND(EXTRACT(EPOCH FROM (pickup_time::TIMESTAMP - order_time::TIMESTAMP)) / 60, 2) AS time_between_order_and_pickup,
	r.duration AS delivery_time,
	ROUND((r.distance / r.duration * 60)::INT, 2) AS avg_speed,
	pizza_num AS pizzas_delivered
FROM ratings AS r
JOIN pizza_runner.customer_orders AS co
	ON r.order_id = co.order_id
JOIN pizza_count_CTE AS pc
	ON r.order_id = pc.order_id
WHERE cancellation IS NULL
```

| customer_id | order_id | runner_id | ratings | order_time          | pickup_time         | time_between_order_and_pickup | delivery_time | avg_speed | pizzas_delivered |
|-------------|----------|-----------|---------|---------------------|---------------------|-------------------------------|---------------|-----------|------------------|
| 101         | 1        | 1         | 5       | 2020-01-01 18:05:02 | 2020-01-01 18:15:34 | 10.53                         | 32            | 38.00     | 1                |
| 101         | 2        | 1         | 5       | 2020-01-01 19:00:52 | 2020-01-01 19:10:54 | 10.03                         | 27            | 44.00     | 1                |
| 102         | 3        | 1         | 5       | 2020-01-02 23:51:23 | 2020-01-03 00:12:37 | 21.23                         | 20            | 40.00     | 2                |
| 102         | 8        | 2         | 5       | 2020-01-09 23:54:33 | 2020-01-10 00:15:02 | 20.48                         | 15            | 94.00     | 1                |
| 103         | 4        | 2         | 5       | 2020-01-04 13:23:46 | 2020-01-04 13:53:03 | 29.28                         | 40            | 35.00     | 3                |
| 104         | 5        | 3         | 5       | 2020-01-08 21:00:29 | 2020-01-08 21:10:57 | 10.47                         | 15            | 40.00     | 1                |
| 104         | 10       | 1         | 5       | 2020-01-11 18:34:49 | 2020-01-11 18:50:20 | 15.52                         | 10            | 60.00     | 2                |
| 105         | 7        | 2         | 5       | 2020-01-08 21:20:29 | 2020-01-08 21:30:45 | 10.27                         | 25            | 60.00     | 1                |

### Question 5: If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
This is very similar to Question 1, but I added the subtraction of $0.3 per kilometer driven. This leaves the company with $73.38.

```sql
SELECT
	'$' || SUM(
		CASE
			WHEN co.pizza_id = 1 THEN (12 - ro.distance * 0.3)
			ELSE (10 - ro.distance * 0.3) END) AS total_income
FROM runner_orders AS ro
JOIN customer_orders AS co
	ON ro.order_id = co.order_id;
```

| total_income |
|----------|
| $73.38        |

### Bonus question: If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?
The two tables to be altered here are `pizza_names` and `pizza_recipes`. For the first, I added the new id and the name, and for the recipes, the new id and all the possible toppings have t obe inserted.

```sql
INSERT INTO pizza_runner.pizza_names
VALUES
(3, 'Supreme')
```
```sql
INSERT INTO pizza_runner.pizza_recipes
VALUES
(3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12')
```

**Week 2 done, now let me go to have some sleep before Week 3!**
