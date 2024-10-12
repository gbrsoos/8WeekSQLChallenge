
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
### Question 1: Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey. Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!
Based on the table attached on the website, I recreated the query pulling the 8 customers of interest. I performed a `JOIN` as it was suggested to see the actual name of the plans the customers indulged in buying. The main thing to mention is that each of the 8 customers started off their career on Foodie-Fi with the trial option, and upgraded afterwards. This is why a `customer_id` is represented multiple times in the database, since each of their purchase (let it be an upgrade or a downgrade) is recorded. You can also see that the supposedly free trial is one week long, since all of them have changed package in one week's time.


```sql
SELECT 
	s.*,
	p.plan_name
FROM subscriptions AS s
JOIN plans AS p
	ON s.plan_id = p.plan_id
WHERE customer_id = 1 
	OR customer_id = 2 
	OR customer_id = 11
	OR customer_id = 13
	OR customer_id = 15 
	OR customer_id = 16
	OR customer_id = 18
	OR customer_id = 19
ORDER BY customer_id, plan_id;
```

| customer_id | plan_id | start_date  | plan_name      |
|-------------|---------|-------------|----------------|
| 1           | 0       | 2020-08-01  | trial          |
| 1           | 1       | 2020-08-08  | basic monthly  |
| 2           | 0       | 2020-09-20  | trial          |
| 2           | 3       | 2020-09-27  | pro annual     |
| 11          | 0       | 2020-11-19  | trial          |
| 11          | 4       | 2020-11-26  | churn          |
| 13          | 0       | 2020-12-15  | trial          |
| 13          | 1       | 2020-12-22  | basic monthly  |
| 13          | 2       | 2021-03-29  | pro monthly    |
| 15          | 0       | 2020-03-17  | trial          |
| 15          | 2       | 2020-03-24  | pro monthly    |
| 15          | 4       | 2020-04-29  | churn          |
| 16          | 0       | 2020-05-31  | trial          |
| 16          | 1       | 2020-06-07  | basic monthly  |
| 16          | 3       | 2020-10-21  | pro annual     |
| 18          | 0       | 2020-07-06  | trial          |
| 18          | 2       | 2020-07-13  | pro monthly    |
| 19          | 0       | 2020-06-22  | trial          |
| 19          | 2       | 2020-06-29  | pro monthly    |
| 19          | 3       | 2020-08-29  | pro annual     |

Click [here](https://github.com/gbrsoos/8WeekSQLChallenge/blob/main/Week3-Foodie_Fi/B.%20Data%20Analysis%20Questions.md) to continue with part B. Data Analysis Questions
