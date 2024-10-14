
## Week 3/B - Data Analysis Questions
![8WeekSQLChallenge - Week3](https://8weeksqlchallenge.com/images/case-study-designs/3.png)

### Introduction
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

### Database
#### Entity Relationship Diagram
![Week 3 - Entity Relationship Diagram](https://8weeksqlchallenge.com/images/case-study-3-erd.png)

This week we are only going to have 2 tables, `plans` and `subscriptions`. As easy as it seems for now, this is surely going to be challenging.

## Solution
### Question 1: How many customers has Foodie-Fi ever had?
To get this number, I used `COUNT(DISTINCT )` to count how many unique `customer_id`s are there in the database.

```sql
SELECT 
	COUNT(DISTINCT customer_id) AS num_of_customers
	FROM subscriptions;
```

|  | num_of_customers |
|--|---------------|
| 1| 1000            |

### Question 2: What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
First, I converted the `start_date` from `DATE` format to characters using `TO_CHAR()`, and obtained only the name of each month. Then I used `COUNT()` to count the occasions where a `trial` plan started for each month and sorted them based on the number.

```sql
SELECT 
    TO_CHAR(start_date::DATE, 'Month') AS month_name,
    COUNT(*) AS total_plans
FROM subscriptions AS s
JOIN plans AS p
	ON p.plan_id = s.plan_id
WHERE s.plan_id = 0
GROUP BY month_name
ORDER BY total_plans DESC;
```

| month_name  | total_plans |
|-------------|-------------|
| March       | 94          |
| July        | 89          |
| January     | 88          |
| May         | 88          |
| August      | 88          |
| September   | 87          |
| December    | 84          |
| April       | 81          |
| October     | 79          |
| June        | 79          |
| November    | 75          |
| February    | 68          |


### Question 3: What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
Even though the question itself is not very clear in terms of what the author is interested in, as per my best understanding, they only want to know the number of purchases per plans from the date 2021. 01. 01. From here on, the query is pretty straightforward, which can be seen below.

```sql
SELECT 
	plan_name,
	COUNT(*) AS num_of_purchase
FROM subscriptions AS s
JOIN plans AS p 
	ON s.plan_id = p.plan_id
WHERE start_date::DATE >= '2021-01-01'
GROUP BY plan_name
ORDER BY num_of_purchase;
```

| plan_name  | num_of_purchase |
|-------------|-------------|
| basic monthly       | 8          |
| pro monthly       | 60          |
| pro annual       | 63          |
| churn       | 71          |

### Question 4: What is the customer count and percentage of customers who have churned rounded to 1 decimal place? 
Initially CTEs came to my mind regarding the way of solution, but I realized that there is no point creating a CTE to perform one single aggregation since I will not be able to join it with the outer query. So I used a nested `SELECT` clause to be able to perform the required division.

```sql
SELECT 
	COUNT(DISTINCT customer_id) AS num_of_churners,
	ROUND(100.0 * COUNT(DISTINCT customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 1) AS perc_of_churners
FROM subscriptions
WHERE plan_id = 4;
```

| num_of_churners  | perc_of_churners |
|-------------|-------------|
| 307      | 30.7          |

### Question 5: How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
To be completely failure-proof, I used 2 CTEs here. One is responsible for creating a row_number for each separate purchase of the customers, and the second one counts whether it is true that the customer started with a trial and followed by churning. In the outer query, I just filter the cases where it is true (`identifier = 2`) and do the necessary calculations.

```sql
WITH presence_cte AS (
	SELECT
		s.*,
		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date)
	FROM subscriptions AS s
	JOIN plans AS p
		ON s.plan_id = p.plan_id
),
identify_cte AS (
	SELECT 
		customer_id,
		COUNT(customer_id) AS identifier
	FROM presence_cte AS p
	WHERE (plan_id = 0 AND row_number = 1) OR (plan_id = 4 AND row_number = 2)
	GROUP BY customer_id
)

SELECT 
	COUNT(customer_id) AS churned_after_trial,
	ROUND(100 * COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 0) AS percent_to_sample
FROM identify_cte AS i
WHERE identifier = 2;
```

| churned_after_trial  | percent_to_sample |
|-------------|-------------|
| 92      | 9        |

### Question 6: What is the number and percentage of customer plans after their initial free trial?
For this solution to work, I had to do a preliminary query, which ensured me about the fact that all the 1,000 customers started their journey with the free trial. Knowing this, I was only interested in knowing the distribution of plans where the `row_number` created in the CTE is `1`.

```sql
WITH second_plan_cte AS (
	SELECT
		s.*,
		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date)
	FROM subscriptions AS s
	JOIN plans AS p
		ON s.plan_id = p.plan_id
)

SELECT 
	plan_name,
	COUNT(plan_name) AS num_after_trial,
	ROUND(100.0 * COUNT(plan_name) / (SELECT COUNT(*) FROM subscriptions), 2) AS perc_after_trial
FROM second_plan_cte AS sp
JOIN plans AS p
	ON sp.plan_id = p.plan_id
WHERE row_number = 2
GROUP BY plan_name;
```

| plan_name      | num_after_trial | perc_after_trial |
|----------------|-----------------|------------------|
| basic monthly  | 546             | 20.60            |
| churn          | 92              | 3.47             |
| pro annual     | 37              | 1.40             |
| pro monthly    | 325             | 12.26            |

### Question 7: What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?


```sql
WITH status_cte AS(
	SELECT
		s.*,
		p.plan_name,
		LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS upcoming_date
	FROM subscriptions AS s
	JOIN plans AS p
		ON s.plan_id = p.plan_id
	WHERE start_date <= '2020-12-31'
)

SELECT 
	plan_name,
	COUNT(DISTINCT customer_id) AS num_cust,
	ROUND(100.0 * COUNT(DISTINCT customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 1) AS perc_cust
FROM status_cte
WHERE upcoming_date IS NULL
GROUP BY plan_name
```

| plan_name      | num_cust | perc_cust |
|----------------|-----------------|------------------|
| basic monthly  | 224             | 22.4            |
| churn  | 236             | 23.6            |
| pro annual  | 195             | 19.5            |
| pro monthly  | 326             | 32.6            |
| trial  | 19             | 1.9            |
