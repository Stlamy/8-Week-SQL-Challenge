	# Case Study #3 - Foodie-Fi
<image src = "https://8weeksqlchallenge.com/images/case-study-designs/3.png" alt = "Image" width = "500" height = "520">

## Index
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Solutions](#solutions)
	- [A. Customer Journey](#customer-journey)
  - [B. Data Analysis Questions](#data-analysis-questions)
  - [C. Challenge Payment Question](#challenge-payment-question)

***

Thanks to Danny for sharing this challenge. You can find the questions and schema used in [this link](https://8weeksqlchallenge.com/case-study-3/).

## Business Task
Danny sees a niche in the streaming service market for a platform that is focused on food-content only. Danny started his startup in 2020 and started selling monethly and annual subscriptions, giving customers unlimited on-demand access to exclusive food videos from around the world.

***

## Entity Relationship Diagram
<image src = "https://user-images.githubusercontent.com/81607668/129744449-37b3229b-80b2-4cce-b8e0-707d7f48dcec.png" alt = "Image" width = "500" height = "260">

Note: the 'churn' plan is an indication that the customer canceled their Foodie-Fi service but will contiunue their plan until the end of the billing period.

***

## Solutions
Execute the queries using PostgreSQL on DB Fiddle. 

***

## Case Study Questions

### A. Customer Journey

**Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.**

````sql
SELECT 
	s.*
	, p.plan_name
FROM foodie_fi.subscriptions s
LEFT JOIN foodie_fi.plans p
ON s.plan_id = p.plan_id
WHERE customer_id = 1
	OR customer_id = 2 
  OR customer_id = 11
  OR customer_id = 13
  OR customer_id = 15
  OR customer_id = 16
  OR customer_id = 18
  OR customer_id = 19
ORDER BY customer_id, start_date;
````

#### Code Explanation
- A more elegant solution would be to use the 'IN' statement within the WHERE condition: 'IN (1, 2, 11, 13, 15, 16, 18, 19);'.


#### Answer

| customer_id | plan_id | start_date               | plan_name     |
| ----------- | ------- | ------------------------ | ------------- |
| 1           | 0       | 2020-08-01T00:00:00.000Z | trial         |
| 1           | 1       | 2020-08-08T00:00:00.000Z | basic monthly |
| 2           | 0       | 2020-09-20T00:00:00.000Z | trial         |
| 2           | 3       | 2020-09-27T00:00:00.000Z | pro annual    |
| 11          | 0       | 2020-11-19T00:00:00.000Z | trial         |
| 11          | 4       | 2020-11-26T00:00:00.000Z | churn         |
| 13          | 0       | 2020-12-15T00:00:00.000Z | trial         |
| 13          | 1       | 2020-12-22T00:00:00.000Z | basic monthly |
| 13          | 2       | 2021-03-29T00:00:00.000Z | pro monthly   |
| 15          | 0       | 2020-03-17T00:00:00.000Z | trial         |
| 15          | 2       | 2020-03-24T00:00:00.000Z | pro monthly   |
| 15          | 4       | 2020-04-29T00:00:00.000Z | churn         |
| 16          | 0       | 2020-05-31T00:00:00.000Z | trial         |
| 16          | 1       | 2020-06-07T00:00:00.000Z | basic monthly |
| 16          | 3       | 2020-10-21T00:00:00.000Z | pro annual    |
| 18          | 0       | 2020-07-06T00:00:00.000Z | trial         |
| 18          | 2       | 2020-07-13T00:00:00.000Z | pro monthly   |
| 19          | 0       | 2020-06-22T00:00:00.000Z | trial         |
| 19          | 2       | 2020-06-29T00:00:00.000Z | pro monthly   |
| 19          | 3       | 2020-08-29T00:00:00.000Z | pro annual    |

---

From the resulting query, observe the onboarding journey for each customer:
- customer_id 1 - started with the 7 day free trial, then downgraded to the basic monthly plan.
- customer_id 2 - started with the 7 day free trial, then upgraded to the annual pro subscription.
- customer_id 11 - started with the 7 day free trial and chose to cancel their subscription to churn.
- customer_id 13 - started with the 7 day free trial, downgraded to the basic monthly, then upgraded back to the pro after over 3 months.
- customer_id 15 - started with the 7 day free trial and mantained their subscription, but then canceled after roughly one month.
- customer_id 16 - started with the 7 day free trial, downgraded to the basic monthly, then upgraded to the annual pro subscription after roughly one month.
- customer_id 18 - started with the 7 day free trial and maintained their subscription.
- customer_id 19 - started with the 7 day free trial, maintained their subscription, then upgraded to the annual pro plan after roughly 2 months.

***

### B. Data Analysis Questions

**1. How many customers has Foodie-Fi ever had?**

````sql
SELECT
	COUNT(DISTINCT customer_id) AS total_customers
FROM foodie_fi.subscriptions;
````

#### Answer

| total_customers |
| --------------- |
| 1000            |

---

Foodie-Fi has 1,000 total customers.

***

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?**

````sql
WITH cte AS (
	SELECT
		EXTRACT(MONTH FROM start_date) AS start_month
	FROM foodie_fi.subscriptions
	WHERE plan_id = 0)
, cte2 AS (
	SELECT
		start_month
		, COUNT(start_month) AS month_count
	FROM cte
	GROUP BY start_month
	ORDER BY start_month
), cte3 AS (
	SELECT
		SUM(month_count) AS total_count
	FROM cte2
), cte4 AS (
	SELECT
		cte2.start_month
  		, cte2.month_count::numeric
    		, cte3.total_count::numeric AS total_count
	FROM cte2, cte3
)

SELECT
	start_month
	, month_count
	, total_count
	, ROUND(month_count / total_count * 100, 2) AS count_share
FROM cte4;
````

| start_month | month_count | total_count | count_share |
| ----------- | ----------- | ----------- | ----------- |
| 1           | 88          | 1000        | 8.80        |
| 2           | 68          | 1000        | 6.80        |
| 3           | 94          | 1000        | 9.40        |
| 4           | 81          | 1000        | 8.10        |
| 5           | 88          | 1000        | 8.80        |
| 6           | 79          | 1000        | 7.90        |
| 7           | 89          | 1000        | 8.90        |
| 8           | 88          | 1000        | 8.80        |
| 9           | 87          | 1000        | 8.70        |
| 10          | 79          | 1000        | 7.90        |
| 11          | 75          | 1000        | 7.50        |
| 12          | 84          | 1000        | 8.40        |

---

#### Code Explanation
While this may not be the most elegant solution, the code is written as follows:
- First CTE limits the [subscriptions] table to only trial subscriptions and extracts all of the start months.
- Second CTE counts all trial subscriptions by start month.
- Third CTE estimates the sum of all trial subscriptions.
- Fourth CTE organizes all of the data estimated in the previous 3 CTEs for use to estimate the share of starting months for all trial subscriptions.

#### Answer
From 1,000 trial subscriptions, the month with the most trial subscriptions was March, while the one with the least was Februrary. 

***

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name. **

````sql
WITH cte AS (
  SELECT
  	*
  	, EXTRACT(MONTH FROM start_date) AS start_month
  FROM foodie_fi.subscriptions
  WHERE start_date >= '2021-01-01'
)

SELECT
	plan_id
    , COUNT(start_month)
FROM cte
GROUP BY plan_id
ORDER BY plan_id;
````

| plan_id | count |
| ------- | ----- |
| 1       | 8     |
| 2       | 60    |
| 3       | 63    |
| 4       | 71    |

---

#### Answer
Limiting the query to subscriptions after the year 2020 (so signups on or after 2021-01-01), observe that there are no plans with id 0, or trial plans. A few explanations as to why could be:
- Foodie-Fi no longer offers the trial option after an initial expansion phase prior to 2021-01-01.
- Given that there are only 1,000 trial plans on record, Foodie-Fi ran a special promotion for 1,000 trial offers for its launch.
- Foodie-Fi is no longer experiencing any user growth and has stopped.

````sql
WITH cte AS (
  SELECT *
  	, EXTRACT(YEAR FROM start_date) AS start_year
  	,  EXTRACT(MONTH FROM start_date) AS start_month
  	,  EXTRACT(DAY FROM start_date) AS start_day
  FROM foodie_fi.subscriptions
), cte2 AS (
  SELECT
	*
  FROM cte
  WHERE plan_id = 0
)

SELECT
	MIN(customer_id) AS first_trial
    , MAX(customer_id) AS last_trial
    , COUNT(customer_id) AS customer_count
    , MIN(start_date) AS first_date
    , MAX(start_date) AS last_date
FROM cte2;
````

Limiting to customers who had a trial subscription, one can observe that customers from 1-1,000 were offered a trial subscription plan. The promotion also lasted from '2020-01-01' to '2020-12-30', so it is unclear whether the subscription was limited to the first year only, or whether it was limited to the first 1,000 customers. It is interesting to note though, that there are only 1,000 unique customers in Foodie-Fi's database, which could actually also suggest that there are no more customers signing up for service. 

| first_trial | last_trial | customer_count | first_date               | last_date                |
| ----------- | ---------- | -------------- | ------------------------ | ------------------------ |
| 1           | 1000       | 1000           | 2020-01-01T00:00:00.000Z | 2020-12-30T00:00:00.000Z |

---

***

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

````sql
WITH cte AS (
  SELECT COUNT(DISTINCT customer_id)::numeric AS churn_count
  FROM foodie_fi.subscriptions
  WHERE plan_id = 4
), cte2 AS (
  SELECT COUNT(DISTINCT customer_id)::numeric AS total_count
  FROM foodie_fi.subscriptions
)

SELECT
	cte.churn_count AS churn_count
	, ROUND(cte.churn_count / cte2.total_count * 100, 1) AS churn_percent
FROM cte, cte2;
````

| churn_count | churn_percent |
| ----------- | ------------- |
| 307         | 30.7          |

---

#### Answer
There are 307 customers that have churned out of 1,000 unique customers, resulting in a percentage of 30.7%.

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

Note: Initially ran a LEFT JOIN two queries filtered for plan_ids either 0 or 4, but this does not account for customers that did sign up for a different plan initially but then canceled. Instead, the DENSE_RANK statement is more suitable to limit to customers that churned immediately after the initial free trial.   

````sql
WITH cte AS (
  SELECT
  	*
  	, DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS date_order
  FROM foodie_fi.subscriptions
), cte2 AS (
  SELECT
  	COUNT(customer_id) AS churn_count
  FROM cte
  WHERE plan_id = 4 AND date_order = 2
), cte3 AS (
  SELECT
  	COUNT(customer_id) AS total_churn
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
)

SELECT
	cte2.churn_count
	, ROUND(cte2.churn_count * 100.0 / cte3.total_churn, 0) AS churn_rate
FROM cte2, cte3;
````

| churn_count | churn_rate |
| ----------- | ---------- |
| 92          | 9          |

---

#### Answer
From a total of 1,000 trial customers, 92 customers directly churned their account.

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

````sql
WITH cte AS (
  SELECT COUNT(DISTINCT customer_id)::numeric AS churn_count
  FROM foodie_fi.subscriptions
  WHERE plan_id = 4
), cte2 AS (
  SELECT COUNT(DISTINCT customer_id)::numeric AS total_count
  FROM foodie_fi.subscriptions
)

SELECT
	cte.churn_count AS churn_count
	, ROUND(cte.churn_count / cte2.total_count * 100, 1) AS churn_percent
FROM cte, cte2;
````

| churn_count | churn_percent |
| ----------- | ------------- |
| 307         | 30.7          |

---

#### Answer
There are 307 customers that have churned out of 1,000 unique customers, resulting in a percentage of 30.7%.

**6. What is the number and percentage of customer plans after their initial free trial?**

````sql
WITH cte AS (
  SELECT
  	s.*
  	, p.plan_name
  	, DENSE_RANK() OVER (PARTITION BY s.customer_id
                         ORDER BY s.start_date) AS date_order
  FROM foodie_fi.subscriptions s
  LEFT JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id
), cte2 AS (
  SELECT
  	plan_id
  	, plan_name
  	, COUNT(plan_id) AS plan_count
  FROM cte
  WHERE date_order = 2
  GROUP BY plan_id, plan_name
  ORDER BY plan_id, plan_name
), cte3 AS (
  SELECT
  	COUNT(customer_id) AS total_trial
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
)

SELECT
	cte2.plan_id
	, cte2.plan_name
	, cte2.plan_count
	, ROUND(cte2.plan_count * 100.0 / cte3.total_trial, 1) AS plan_rates
FROM cte2, cte3
ORDER BY plan_rates DESC;
````

| plan_id | plan_name     | plan_count | plan_rates |
| ------- | ------------- | ---------- | ---------- |
| 1       | basic monthly | 546        | 54.6       |
| 2       | pro monthly   | 325        | 32.5       |
| 4       | churn         | 92         | 9.2        |
| 3       | pro annual    | 37         | 3.7        |

---

#### Answer
The basic monthly was the most popular choice right after the initial trial period at 54.6%, followed by the pro monthly at 32.5%. The least popular plan was the pro annual at 3.7%.

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

````sql
WITH cte AS (
  SELECT
  	s.*
  	, p.plan_name
  	, DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.start_date DESC) AS date_order
  FROM foodie_fi.subscriptions s
  LEFT JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id
  WHERE start_date <= '2020-12-31'
), cte2 AS (
  SELECT
  	plan_name
  	, COUNT(plan_name) AS plan_count
  FROM cte
  WHERE date_order = 1
  GROUP BY plan_name
  ORDER BY plan_name
), cte3 AS (
  SELECT
  	SUM(plan_count) AS total_plans
  FROM cte2
)

SELECT
	plan_name
	, plan_count
	, ROUND(plan_count * 100.0 / total_plans, 1) AS plan_rate
FROM cte2, cte3
ORDER BY plan_count DESC;
````

| plan_name     | plan_count | plan_rate |
| ------------- | ---------- | --------- |
| pro monthly   | 326        | 32.6      |
| churn         | 236        | 23.6      |
| basic monthly | 224        | 22.4      |
| pro annual    | 195        | 19.5      |
| trial         | 19         | 1.9       |

---

#### Code Explanation
The initial CTE orders all customers based on their latest start dates in order to account for the subscription characteristics; upgrading plans go into effect immediately, while downgrading a plan will maintain the higher plan until the plan end date. As such, taking the last plan before 2020-12-31 will collect the customer plans as of that date. 

#### Answer
Most customers held the pro-monthly plan as of 2020-12-31 at 32.6%, while the next most frequent plan churn at 23.6%. The least held plan was the trial, where only 1.9% of customers remained on that particular plan.
