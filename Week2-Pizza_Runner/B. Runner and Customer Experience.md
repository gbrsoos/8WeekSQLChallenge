## Week 2/B - Customer and Runner Experience
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

### Question 1: How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
The first question of the second part starts off strong. Since PostgreSQL follows ISO8601 when it comes to dates which defines the first week of the year as the week that contains the first Thursday of the year. To tackle this, I figured the method `FLOOR((registration_date - DATE '2021-01-01') / 7) + 1` which takes the difference between the actual `registration_date` and the first day of the year, divides it by 7 to convert it to weeks, uses the `FLOOR()` function which takes the largest smaller or equal integer, and adds +1 for the proper numbering. Let us take an example, `2021-01-06`. The difference is 5 days, which is 5/7 after the division. `FLOOR()` then returns 0, since that is the closest smaller integer, and adds 1. Thus `2021-01-06` is the last day of week `1`, since 7/7 = 1, `FLOOR(1) = 1` and 1+1 = 2.

```sql
SELECT 
	FLOOR((registration_date - DATE '2021-01-01') / 7) + 1 AS week_number,
	COUNT(runner_id) AS runners
FROM runners
GROUP BY FLOOR((registration_date - DATE '2021-01-01') / 7) + 1
ORDER BY week_number ASC;

```

|  | week_number | runners |
|--|--------------|---------|
| -| 1           | 2         |
| -| 2           |  1        |
| -| 3           |   1       |

### Question 2: What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
Seems like this part is heavily going to focus on date and/or distance-related measures. Here, I joined the well-known `runner_orders` and `customer_orders` tables to obtain both the `pickup_time` and the `order_time`. After that, I performed a series of functions: after subtracting the order time from the pickup time (to get the difference between the two), I used `EXTRACT(EPOCH FROM...)` To capture the entire difference in seconds, and the divided it by 60 to get the value in minutes. `AVG()` is responsible for calculating the mean and `ROUND(x, 2)` limits the number of decimals.

```sql
SELECT
    ROUND(AVG(EXTRACT(EPOCH FROM (pickup_time::TIMESTAMP - order_time) / 60)), 2) AS minutes_to_pickup
FROM runner_orders AS ro
JOIN customer_orders AS co
	ON ro.order_id = co.order_id;
```

|  | minutes_to_pickup |
|--|---------------|
| 1| 18.59            |

### Question 3: Is there any relationship between the number of pizzas and how long the order takes to prepare?
To answer this, I broke the solution down into two steps. In the first one, I created a CTE which includes the number of pizzas ordered for each order and the time it has taken to prepare the order. In the outer query, I average these preparation times for each unique number of ordered pizzas. Regarding the relationship, I created an additional column calculating the time/pizza ratio. Here, it is clear that preparing one pizza takes significantly more time if there is only one pizza in the order than in the cases with more pizzas included. This might be due to the human factor, since people tend to think they have more time due to the order not being so stacked.

```sql
WITH per_order_CTE AS(
	SELECT
		COUNT(pizza_id) AS pizza_num,
		ROUND(EXTRACT(EPOCH FROM (pickup_time::TIMESTAMP - order_time) / 60), 2) AS minutes_to_pickup
	FROM customer_orders AS co
	JOIN runner_orders AS ro
		ON co.order_id = ro.order_id
	WHERE pickup_time IS NOT NULL
	GROUP BY ro.pickup_time, co.order_time
)

SELECT 
	pizza_num,
	ROUND(AVG(minutes_to_pickup), 2),
	ROUND(AVG(minutes_to_pickup) / pizza_num, 2) AS time_per_pizza
FROM per_order_CTE
GROUP BY pizza_num
ORDER BY pizza_num;
```

|  | pizza_num | avg_preparation_time  | time_per_pizza|
|--|--------------|---------------|------------------|
| 1| 1            | 12.36         | 12.36         |
| 2| 2           | 18.38         | 9.19         |
| 3| 3           | 29.28         | 9.76          |

### Question 4: What was the average distance travelled for each customer?
The only detail worth mentioning in this query is that originally the distance column's data type is `DOUBLE PRECISION`, so I had to temporarily transform it to `NUMERIC`, so that the `ROUND()` function can handle it properly.


```sql
SELECT
	customer_id,
	ROUND(AVG(distance)::NUMERIC ,2) AS avg_distance
FROM customer_orders AS co
JOIN runner_orders AS ro
	ON co.order_id = ro.order_id
WHERE pickup_time IS NOT NULL
GROUP BY customer_id
ORDER BY customer_id;
```

|  | customer_id | avg_distance  |
|--|--------------|---------------|
| 1| 101        | 20.00             |
| 2| 102       | 16.73             |
| 3| 103      | 23.40            |
| 4| 104     | 10.00             |
| 5| 105      | 25.00             |

### Question 5: What was the difference between the longest and shortest delivery times for all orders?


```sql
SELECT
	MAX(duration::NUMERIC) - MIN(duration::NUMERIC) AS time_difference
FROM runner_orders
WHERE pickup_time IS NOT NULL;
```

|  | time_difference  |
|--|-------------|
| 1| 30           |

### Question 6: What was the average speed for each runner for each delivery and do you notice any trend for these values?
To calculate the average speed, I divided the `distance` and the `duration` columns, which leaves me speed in km/m. To transfer this to km/h, the values need to be multiplied by 60. Regarding the trends, the fluctuation of average speed for the runner with `runner_id = 2` is suspiciously big, since the difference between `35.40` and `93.60` is almost 300%.

```sql
SELECT
	runner_id,
	ro.order_id,
	ROUND((distance::NUMERIC / duration::NUMERIC), 2) * 60 AS avg_speed
FROM customer_orders AS co
JOIN runner_orders AS ro
	ON co.order_id = ro.order_id
WHERE pickup_time IS NOT NULL
GROUP BY runner_id, ro.order_id, ro.distance, ro.duration
ORDER BY runner_id, order_id;
```

|  | runner_id  | order_id | avg_speed  |
|--|--------------|-------|-------|
| 1| 1	          | 1  | 37.80  |
| 2| 1	          | 2 | 44.40  |
| 3| 1	          | 3 |  40.20 |
| 4| 1	          | 10 |  60.00 |
| 5| 2	          | 4 | 35.40  |
| 6| 2	          | 7 | 60.00  |
| 7| 2	          | 8 |  93.60 |
| 8| 3	          | 5 |  40.20 |

### Question 7: What is the successful delivery percentage for each runner?
For the `success_rate` calculation I created a `CASE` statement, where 1 represents a successful delivery and 0 represents the unsuccessful ones. I multiplied the values by 100 and divided the result with all the orders a runner had (`COUNT(runner_id`).

```sql
SELECT
	runner_id,
	(100 * SUM(CASE 
			WHEN cancellation IS NULL THEN 1 ELSE 0 END) / COUNT(runner_id)) AS succes_rate
FROM runner_orders
GROUP BY runner_id
ORDER BY runner_id;
```

|  | runner_id  | success_rate  |
|--|--------------|-----------|
| 1| 1	          | 100         |
| 2| 2	          | 75       |
| 3| 3	          | 50        |

Click [here](https://github.com/gbrsoos/8WeekSQLChallenge/blob/main/Week2-Pizza_Runner/C.%20Ingredient%20Optimisation.md) to continue with C. Ingredient Optimisation.
