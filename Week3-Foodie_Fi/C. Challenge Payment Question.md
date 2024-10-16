
## Week 3/C - Challenge Payment Question 
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
### Question 1: The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
### - monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
### - upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
### - upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
### - once a customer churns they will no longer make payments 
The key of this query is the `generate_series()` command, which creates series based on a start date, an end date, and a given interval. In our case, I had to use `COALESCE()` to identify the two possible end date options, whih is either the date when the given ustomer switched plan, or the last day of 2020 which was determined in the question.

```sql
CREATE TABLE payments AS
WITH base_cte AS (	
	SELECT
		s.customer_id,
		s.plan_id,
		p.plan_name,
		s.start_date,
		p.price,
		LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS switch_date,
		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rank_num
	FROM subscriptions AS s
	JOIN plans AS p
		ON s.plan_id = p.plan_id
	ORDER BY customer_id, s.plan_id
),
payments_cte AS (
    SELECT
        b.customer_id,
        b.plan_id,
        b.plan_name,
        b.price,
        generate_series(
            b.start_date,
            COALESCE(b.switch_date - interval '1 day', '2020-12-31'::DATE),
            interval '1 month')::DATE AS payment_date,
    	ROW_NUMBER() OVER (PARTITION BY b.customer_id 
			ORDER BY generate_series(
			b.start_date, 
			COALESCE(b.switch_date::DATE - interval '1 day', '2020-12-31'::DATE),
			interval '1 month')) AS payment_order
	FROM base_cte AS b
    WHERE b.plan_name != 'churn'  
)
SELECT 
    customer_id,
    plan_id,
    plan_name,
    payment_date,
    price AS amount,
    payment_order
FROM payments_cte
ORDER BY customer_id, payment_date;
```

| customer_id | plan_id | plan_name      | payment_date | amount | payment_order |
|-------------|---------|----------------|--------------|--------|---------------|
| 1           | 0       | trial          | 2020-08-01   | 0.00   | 1             |
| 1           | 1       | basic monthly  | 2020-08-08   | 9.90   | 2             |
| 1           | 1       | basic monthly  | 2020-09-08   | 9.90   | 3             |
| 1           | 1       | basic monthly  | 2020-10-08   | 9.90   | 4             |
| 1           | 1       | basic monthly  | 2020-11-08   | 9.90   | 5             |
| 1           | 1       | basic monthly  | 2020-12-08   | 9.90   | 6             |
| 2           | 0       | trial          | 2020-09-20   | 0.00   | 1             |
| 2           | 3       | pro annual     | 2020-09-27   | 199.00 | 2             |
| 2           | 3       | pro annual     | 2020-10-27   | 199.00 | 3             |
| 2           | 3       | pro annual     | 2020-11-27   | 199.00 | 4             |
| 2           | 3       | pro annual     | 2020-12-27   | 199.00 | 5             |
| 3           | 0       | trial          | 2020-01-13   | 0.00   | 1             |
| 3           | 1       | basic monthly  | 2020-01-20   | 9.90   | 2             |
| 3           | 1       | basic monthly  | 2020-02-20   | 9.90   | 3             |
| 3           | 1       | basic monthly  | 2020-03-20   | 9.90   | 4             |
| 3           | 1       | basic monthly  | 2020-04-20   | 9.90   | 5             |
| 3           | 1       | basic monthly  | 2020-05-20   | 9.90   | 6             |
| 3           | 1       | basic monthly  | 2020-06-20   | 9.90   | 7             |
| 3           | 1       | basic monthly  | 2020-07-20   | 9.90   | 8             |
| 3           | 1       | basic monthly  | 2020-08-20   | 9.90   | 9             |

