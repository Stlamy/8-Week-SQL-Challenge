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
SELECT
	EXTRACT(MONTH from
FROM foodie_fi.subscriptions;
````

#### Answer

| total_customers |
| --------------- |
| 1000            |

---

Foodie-Fi has 1,000 total customers.

***
