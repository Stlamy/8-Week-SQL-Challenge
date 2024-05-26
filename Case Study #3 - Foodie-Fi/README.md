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

**8. How many customers have upgraded to an annual plan in 2020?**

````sql
WITH cte1 AS (
  SELECT
  	*
  	, DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS sign_order
  FROM foodie_fi.subscriptions
  WHERE EXTRACT(YEAR FROM start_date) = 2020
)

SELECT
  	COUNT(DISTINCT customer_id) AS annual_count
FROM cte1
WHERE plan_id = 3 AND sign_order >= 2;
````

| annual_count |
| ------------ |
| 195          |

---

#### Answer
There were 195 customers that have upgraded to the annual plan in the year 2020.

***

**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**

````sql
WITH cte1 AS (
  SELECT
  	*
  	, DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS sign_order
  FROM foodie_fi.subscriptions
), cte2 AS (
  SELECT
  	*
  FROM cte1
  WHERE sign_order = 1
), cte3 AS (
  SELECT
  	*
  FROM cte1
  WHERE plan_id = 3
)

SELECT
  	ROUND(AVG(cte3.start_date - cte2.start_date), 0) AS avg_days
FROM cte3
LEFT JOIN cte2
ON cte3.customer_id = cte2.customer_id;
````

| avg_days |
| -------- |
| 105      |

---

#### Answer
Given the population of customers that have upgraded to the annual plan, it took them an average of 105 days to transition to the annual plan.

***

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**

````sql
WITH cte1 AS (
  SELECT
  	*
  	, DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS sign_order
  FROM foodie_fi.subscriptions
), cte2 AS (
  SELECT
  	*
  FROM cte1
  WHERE sign_order = 1
), cte3 AS (
  SELECT
  	ROUND(cte1.start_date - cte2.start_date, 0) AS day_difference
  	, CASE WHEN
  		ROUND(cte1.start_date - cte2.start_date, 0) <= 30
  	THEN '0-30 days'
  	WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 60
  	THEN '31-60 days'
	WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 90
  	THEN '61-90 days'
  	WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 120
  	THEN '91-120 days'
    WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 150
  	THEN '121-150 days'
    WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 180
  	THEN '151-180 days'
    WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 210
  	THEN '181-210 days'
    WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 240
  	THEN '211-240 days'
    WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 270
  	THEN '241-270 days'
    WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 300
  	THEN '271-300 days'
    WHEN ROUND(cte1.start_date - cte2.start_date, 0) <= 330
  	THEN '301-330 days'
  	ELSE '> 330 days' END AS day_bracket
  FROM cte1
  LEFT JOIN cte2
  ON cte1.customer_id = cte2.customer_id
  WHERE cte1.plan_id = 3
)

SELECT
  	day_bracket
    , COUNT(day_bracket) AS customer_count
    , ROUND(AVG(day_difference), 0) AS avg_days
FROM cte3
GROUP BY day_bracket
ORDER BY day_bracket;
````

| day_bracket  | customer_count | avg_days |
| ------------ | -------------- | -------- |
| 0-30 days    | 49             | 10       |
| 121-150 days | 42             | 133      |
| 151-180 days | 36             | 162      |
| 181-210 days | 26             | 191      |
| 211-240 days | 4              | 224      |
| 241-270 days | 5              | 257      |
| 271-300 days | 1              | 285      |
| 301-330 days | 1              | 327      |
| 31-60 days   | 24             | 42       |
| 61-90 days   | 34             | 71       |
| 91-120 days  | 35             | 101      |
| > 330 days   | 1              | 346      |

---

#### Code Explanation
- Am sure there is a more elegant way to set up the bins, but the table above successfully separates out the 105 average estimated in question 9.


***

**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**

````sql
WITH cte1 AS (
  SELECT
  	*
  	, DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS sign_order
  FROM foodie_fi.subscriptions
  WHERE EXTRACT(YEAR FROM start_date) = 2020 AND plan_id IN (1, 2)
)

SELECT
  	COUNT(customer_id) AS downgrade_count
FROM cte1
WHERE plan_id = 1 AND sign_order = 2;
````

| downgrade_count |
| --------------- |
| 0               |

---

#### Answer
- There are no customers that downgraded from the 'Pro Monthly' plan to the 'Basic Monthly' plan in 2020.

***

### C. Challenge Payment Question

- Currently cannot think of an elegant way to create a table using SQL.
- Want to use the 'LEAD' statement to create another column with information in other rows of the same customer_id.

### D. Outside the Box Questions

**1. How would you calculate the rate of growth for Foodie-Fi?**
- For a relatively new startup like Foodie-Fi, a good way to measure growth would be to observe the month-over-month change in a few potential metrics such as revenue or new customers.

**2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?**
- For a relatively new startup like Foodie-Fi, a good way to measure growth would be to observe the month-over-month change in a few potential metrics such as revenue or new customers. Since Foodie-Fi implements a business model that depends on customer subscriptions, it makes sense that an increase in new customers represents new sources of revenue, so looking at how many customers sign up for the service month by month seems to be a reasonable metric to review.

**3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?**
- Since Foodie-Fi implements a subscription model, it's important to ensure that customers stay on the more profitable services. As such, movement from the premium models to the basic subscriptions and understanding why that happens would be an interesting question to answer. On the other hand, there is no information on how much each plan costs Foodie-Fi to service. This is something that Danny could implement into his data collection process to better understand Foodie-Fi's profitablility.

**4. f the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?**
- General demographic questions are always helpful to understand, so including questions that ask the customer to identify their age and gender would be helpful.
	- What is your gender?
	- What is your age?
 - Some other questions that are more specific to Foodie-Fi are:
	- What did you like about Foodie-Fi? One could include a selection of potential answers on some of the supposed strong points of Foodie-Fi's service, such as ease of use or availability of content, as well as a type box for other reasons not listed. 
 	- Why are you choosing to cancel your subscription? Like the above bullet, potential answers may be price, availability of content, etc.

**5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?**
- A potential tool Danny could use is to provide a special offer that discounts the price of a subscription by xx% compared to a regular subscription for the same number of months.
	- To evaluate this 'lever', one could compare the number of customers that churn before and after the promotion was offered. Another option for testing would be to conduct A/B testing and test the impact of the promotion. This can be done through a simple logistic regression model (since there are two outcomes: continue to subscribe or churn), or through a different classification model such as decision trees.
 - A different tool would be to understand whether there are substitutes for Foodie-Fi. For instance, Netflix and other services also provide a significant number of food-focused shows conveniently, along with other content. If one of the major reasons why customers chose to churn was the lack of content on Foodie-Fi (only wanted to watch a particular series, did not update list of content frequently enough, etc.), an option would be to decrease the price of all subscriptions and downsize.

