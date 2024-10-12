
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
	FROM subscriptions
```

|  | num_of_customers |
|--|---------------|
| 1| 1000            |

### Question 2:What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
***DOLGOK IDE***

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


