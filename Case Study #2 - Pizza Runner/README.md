# Case Study #2 - Pizza Runner
<image src = "https://8weeksqlchallenge.com/images/case-study-designs/2.png" alt = "Image" width = "500" height = "520">

## Index
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Solutions](#solutions)

***

Thanks to Danny for sharing this challenge. You can find the questions and schema used in [this link](https://8weeksqlchallenge.com/case-study-2/).

## Business Task
Danny needs initial seed funding to start a new "Pizza Empire". But since he isn't able to earn all of the seed funding by selling pizza alone, he combined it with an "Uber" delivery service alongside the pizza store. Danny recruited "runners" to deliver pizza from headquarters and maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers. Danny has provided us with the following six tables to answer his questions: runner_orders, runners, customer_orders, pizza_names, pizza_recipes, and pizza_toppings.

***

## Entity Relationship Diagram
<image src = "https://private-user-images.githubusercontent.com/81607668/242152356-78099a4e-4d0e-421f-a560-b72e4321f530.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MTUyNzg2NzEsIm5iZiI6MTcxNTI3ODM3MSwicGF0aCI6Ii84MTYwNzY2OC8yNDIxNTIzNTYtNzgwOTlhNGUtNGQwZS00MjFmLWE1NjAtYjcyZTQzMjFmNTMwLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA1MDklMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwNTA5VDE4MTI1MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWU5N2I5OGYzNDk0YzY4MWZmYTA3ZTc4MzQyOTc1NWZmZTRmNTVjMDg0NThkOWZiZjAzYjNhOGMyY2JlMTNjMTQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.QBvPtP_jpuzIjzr5Zkf9kjuTg5t65TuBR-RczTa8vQg" alt = "Image" width = "500" height = "260">

***

## Solutions
Execute the queries using PostgreSQL on DB Fiddle. 

***

## Data Review and Cleaning
Notice the following characteristics explained in the case study:
- Table 2: customer_orders
    - Customers can order multiple pizzas in a single order with varying exclusions and extras

There are a few tables that seem to handle data differently. Listed on the case study webpage:
- Table 2: customer_orders
    - exclusions and extras handle missing values differently. Notice ' ' and 'null' in customer_orders.exclusions and 'NaN', ' ', and 'null' in customer_orders.extras.
- Table 3: runner_orders
    - runner_orders.distance has a mix of string and integer observations, such as '20km' and '10'.
    - runner_orders.duration has the same issue as above, as well as inconsistently handling null values. Examples include '32 minutes', '20 mins', '40', and 'null'.
    - runner_orders.cancellation is inconsistent in handling null values. Observations include ' ', 'NaN', and 'null'.


To clean the 'customer_orders' table:
````sql
CREATE TEMP TABLE customer_orders_fix AS
  SELECT
    order_id
    , customer_id
    , pizza_id
    , CASE
    	WHEN exclusions = ''
        	OR exclusions = 'null' THEN NULL
        	ELSE exclusions
        END AS exclusions
    , CASE
    	WHEN extras = ''
        	OR extras = 'NaN' 
            OR extras = 'null' THEN NULL
        	ELSE extras
        END AS extras
    , order_time
   FROM pizza_runner.customer_orders;
````

To clean the 'runner_orders' table:
````sql
CREATE TEMP TABLE runner_orders_fix AS
  SELECT
    order_id
    , runner_id
    , pickup_time
    , CASE
    	WHEN distance LIKE '%km%'
        	THEN REPLACE(distance, 'km', '')
        WHEN distance LIKE '%kms%'
        	THEN REPLACE (distance, 'kms', '')
        ELSE distance END AS distance
    , CASE
    	WHEN duration LIKE '%minutes%'
        	THEN REPLACE(duration, 'minutes', '')
        WHEN duration LIKE '%minute'
        	THEN REPLACE(duration, 'minute', '')
        WHEN duration LIKE '%mins%'
        	THEN REPLACE(duration, 'mins', '')
        WHEN duration LIKE 'null'
        	THEN NULL
        ELSE duration END AS duration
    , CASE
    	WHEN cancellation = '' OR cancellation = 'NaN' OR cancellation = 'null' THEN NULL
        ELSE cancellation
        END AS cancellation
   FROM pizza_runner.runner_orders;
````

***

## Case Study Questions

Note: temp tables should be pasted into Query window before running the code below.

### A. Pizza Metrics

**1. How many pizzas were ordered?**

````sql
SELECT COUNT(order_id) AS pizzas_ordered
FROM customer_orders_fix;
````

#### Answer

| pizzas_ordered |
| -------------- |
| 14             |

---

From the resulting query, there were a total of 14 pizzas ordered.

***

**2. How many unique customer orders were made?**

````sql
SELECT
	COUNT(DISTINCT order_id) AS unique_cust_orders
FROM customer_orders_fix;
````

#### Code Explanation
- Use new statement DISTINCT to count unique order_ids.

#### Answer

| unique_cust_orders |
| ------------------ |
| 10                 |

---

From the resulting query, there were a total of 5 unique customer orders made.

***

**3. How many successful orders were delivered by each runner?**

````sql
SELECT
    runner_id
    , COUNT(order_id) AS successful_order
FROM runner_orders_fix
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY runner_id;
````

#### Answer

| runner_id | successful_order |
| --------- | ---------------- |
| 1         | 4                |
| 2         | 3                |
| 3         | 1                |

---

From the resulting query, there were a total of 8 successful runner orders.
- Runner 1 made 4 succesful orders.
- Runner 2 made 3 succesful orders.
- Runner 3 made 1 succesful orders.

***

**4. How many of each type of pizza was delivered?**

````sql
SELECT
    pizza_names.pizza_name
    , COUNT(customer_orders_fix.pizza_id) AS successful_order
FROM runner_orders_fix
LEFT JOIN customer_orders_fix
ON customer_orders_fix.order_id = runner_orders_fix.order_id
LEFT JOIN pizza_runner.pizza_names
ON customer_orders_fix.pizza_id = pizza_names.pizza_id
WHERE cancellation IS NULL
GROUP BY pizza_names.pizza_name
ORDER BY pizza_names.pizza_name;
````

#### Answer

| pizza_name | successful_order |
| ---------- | ---------------- |
| Meatlovers | 9                |
| Vegetarian | 3                |

---

From the resulting query, there were a total of 12 pizzas that were successfully delivered.
- 9 Meatlovers pizzas were successful delivered.
- 3 Vegetarian pizzas were successful delivered.

***

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

````sql
WITH count_meat AS (
	SELECT
		customer_id
  		, COUNT(pizza_id) AS orders_meat
  	FROM customer_orders_fix
  	WHERE pizza_id = 1
	GROUP BY customer_id)
    , count_veg AS (
	SELECT
  		customer_id
  		, COUNT(pizza_id) AS orders_veg
  	FROM customer_orders_fix
  	WHERE pizza_id = 2
	GROUP BY customer_id)

SELECT DISTINCT
    count_meat.customer_id
    , count_meat.orders_meat
    , count_veg.orders_veg
FROM count_meat
LEFT JOIN count_veg
ON count_meat.customer_id = count_veg.customer_id;
````

#### Answer

| customer_id | orders_meat | orders_veg |
| ----------- | ----------- | ---------- |
| 101         | 2           | 1          |
| 102         | 2           | 1          |
| 103         | 3           | 1          |
| 104         | 3           | null       |
| 105         | null        | 1          |

---

From the resulting query, observe the following pizza orders by customer.
- Customer 101 ordered 2 Meatlovers and 1 Vegetable.
- Customer 102 ordered 2 Meatlovers and 1 Vegetable.
- Customer 103 ordered 3 Meatlovers and 1 Vegetable.
- Customer 104 ordered 3 Meatlovers and no Vegetable.

***

**6. What was the maximum number of pizzas delivered in a single order?**

````sql

````

#### Answer

---

From the resulting query, observe a max order of 3 pizzas from a single order.

***

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

````sql
SELECT
    customer_id
    , SUM(CASE 
        WHEN exclusions IS NOT NULL OR extras IS NOT NULL 
        THEN 1
		ELSE 0 END) AS pizza_changes
    , SUM(CASE 
          WHEN exclusions IS NULL AND extras IS NULL 
          THEN 1
		ELSE 0 END) AS pizza_nochanges
FROM customer_orders_fix
GROUP BY customer_id
ORDER BY customer_id;
````

#### Answer

| customer_id | pizza_changes | pizza_nochanges |
| ----------- | ------------- | --------------- |
| 101         | 0             | 3               |
| 102         | 0             | 3               |
| 103         | 4             | 0               |
| 104         | 2             | 1               |
| 105         | 1             | 0               |

---

From the resulting query, there are 7 pizzas with changes and 7 with no changes. Specifically:
- Customer 101 ordered 0 pizzas with changes and 3 pizzas with no changes.
- Customer 102 ordered 0 pizzas with changes and 3 pizzas with no changes.
- Customer 103 ordered 4 pizzas with changes and 0 pizzas with no changes.
- Customer 104 ordered 2 pizzas with changes and 1 pizzas with no changes.
- Customer 105 ordered 1 pizzas with changes and 0 pizzas with no changes.

***

**8. How many pizzas were delivered that had both exclusions and extras?**

````sql
WITH total_count AS (
  SELECT	customer_orders_fix.*
		, CASE
			WHEN customer_orders_fix.extras IS NOT NULL
			AND customer_orders_fix.exclusions IS NOT NULL
			THEN 1
			ELSE 0 END AS all_change
  FROM customer_orders_fix
  LEFT JOIN runner_orders_fix
  ON customer_orders_fix.order_id = runner_orders_fix.order_id
  WHERE runner_orders_fix.cancellation IS NULL)
  
SELECT	SUM(all_change) AS total_orders
FROM total_count;
````

#### Answer

| total_orders |
| ------------ |
| 1            |

---

From the resulting query, there is 1 successfully delivered pizza that has both extra and exceptions made to the order.

***

**9. What was the total volume of pizzas ordered for each hour of the day?**

````sql
SELECT	EXTRACT(HOUR FROM order_time) as order_hour
	, COUNT(order_id) AS hourly_orders
FROM customer_orders_fix
GROUP BY order_hour
ORDER BY order_hour;
````

#### Answer

| order_hour | hourly_orders |
| ---------- | ------------- |
| 11         | 1             |
| 13         | 3             |
| 18         | 3             |
| 19         | 1             |
| 21         | 3             |
| 23         | 3             |

---

From the resulting query, observe the following orders made by hour:
- There are 1 pizzas ordered at hour 11.
- There are 3 pizzas ordered at hour 13.
- There are 3 pizzas ordered at hour 18.
- There are 1 pizzas ordered at hour 19.
- There are 3 pizzas ordered at hour 21.
- There are 3 pizzas ordered at hour 23.

***

**10. What was the volume of orders for each day of the week?**

````sql
SELECT	to_char(order_time, 'Day') as order_day
		, COUNT(order_id) AS orders_by_day
FROM customer_orders_fix
GROUP BY order_day
ORDER BY order_day;
````

#### Answer

| order_day | orders_by_day |
| --------- | ------------- |
| Friday    | 1             |
| Saturday  | 5             |
| Thursday  | 3             |
| Wednesday | 5             |

---

From the resulting query, observe the following orders made by day of the week:
- There are 1 pizzas ordered on Fridays.
- There are 5 pizzas ordered on Saturdays.
- There are 3 pizzas ordered on Thursdays.
- There are 5 pizzas ordered on Wednesdays.

***

### B. Runner and Customer Experience

**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

````sql
SELECT	DATE_PART('week', runners.registration_date) AS week_num
        , COUNT(runners.runner_id) AS runners_sign_up
FROM pizza_runner.runners
GROUP BY week_num;
````

#### Answer

| week_num | runners_sign_up |
| -------- | --------------- |
| 53       | 2               |
| 1        | 1               |
| 2        | 1               |

---

From the resulting query, observe the runner registrations for each week:
- There are 2 runners that registered on the 53rd week.
- There is 1 runner that registered on the 1st week.
- There is 1 runner that registered on the 2nd week.

***

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

````sql
WITH pickup_time AS (
  SELECT runner_id
  		 , order_id
  		 , pickup_time::int
  FROM runner_orders_fix
  WHERE pickup_time IS NOT NULL)

SELECT	pickup_time.runner_id
		, AVG(pickup_time.pickup_time - customer_orders_fix.order_time) AS avg_minutes
FROM pickup_time
LEFT JOIN customer_orders_fix
ON pickup_time.order_id = customer_orders_fix.order_id
GROUP BY runner_id
ORDER BY runner_id;
````

#### Answer

| week_num | runners_sign_up |
| -------- | --------------- |
| 53       | 2               |
| 1        | 1               |
| 2        | 1               |

---

From the resulting query, observe the runner registrations for each week:
- There are 2 runners that registered on the 53rd week.
- There is 1 runner that registered on the 1st week.
- There is 1 runner that registered on the 2nd week.

***

