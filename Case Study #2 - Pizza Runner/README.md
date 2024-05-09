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
    	WHEN exclusions = '' THEN NULL
        	ELSE exclusions
        END AS exclusions
    , CASE
    	WHEN extras = ''
        	OR extras = 'NaN' THEN NULL
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
