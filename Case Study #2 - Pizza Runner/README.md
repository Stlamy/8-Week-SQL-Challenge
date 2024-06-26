# Case Study #2 - Pizza Runner
<image src = "https://8weeksqlchallenge.com/images/case-study-designs/2.png" alt = "Image" width = "500" height = "520">

## Index
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Solutions](#solutions)
	- [A. Pizza Metrics](#pizza-metrics)
   	- [B. Runner and Customer Experience](#runner-customer-experience)
   	- [C. Ingredient Optimisation](#ingredient-optimisation)
   	- [D. Pricing and Ratings](#pricing-ratings)

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
    , CASE
    	WHEN pickup_time LIKE 'null'
        	THEN NULL
        ELSE pickup_time END AS pickup_time
    , CASE
    	WHEN distance LIKE '%km%'
        	THEN REPLACE(distance, 'km', '')
        WHEN distance LIKE '%kms%'
        	THEN REPLACE (distance, 'kms', '')
        WHEN distance LIKE 'null'
        	THEN NULL
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
WITH cte AS (
  SELECT c.*
  FROM customer_orders_fix c
  LEFT JOIN runner_orders_fix r
  ON c.order_id = r.order_id
  WHERE r.cancellation IS NULL)

SELECT
	order_id
    , COUNT(order_id) AS max_pizzas
FROM cte
GROUP BY order_id
ORDER BY max_pizzas DESC
LIMIT 1;
````

#### Answer

| order_id | max_pizzas |
| -------- | ---------- |
| 4        | 3          |

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
  SELECT runner_orders_fix.runner_id
	 , runner_orders_fix.order_id
  	 , runner_orders_fix.pickup_time::timestamp AS pickup_time
  	 , customer_orders_fix.order_time::timestamp AS order_time
  FROM runner_orders_fix
  LEFT JOIN customer_orders_fix
  ON runner_orders_fix.order_id = customer_orders_fix.order_id
  WHERE pickup_time IS NOT NULL
)

SELECT	runner_id
	, ROUND(AVG(EXTRACT(EPOCH FROM pickup_time - order_time)/60)::numeric, 2) AS avg_minutes
FROM pickup_time
GROUP BY runner_id
ORDER BY runner_id;
````

#### Code Explanation
- Use new statement 'EXTRACT(EPOCH))' to extract the timestamp in terms of seconds. By subtracting the difference between order time and pick up time in seconds, then dividing by 60 to estimate the difference in terms of minutes, the result is more precise. 


#### Answer

| runner_id | avg_minutes |
| --------- | ----------- |
| 1         | 15.68       |
| 2         | 23.72       |
| 3         | 10.47       |

---

From the resulting query, it takes an average of:
- 15.68 minutes for runner ID 1.
- 23.72 minutes for runner ID 2.
- 10.47 minutes for runner ID 3.

***

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

````sql
WITH pizza_count AS (
  SELECT customer_orders_fix.order_id
  	 , COUNT(customer_orders_fix.order_id) AS num_pizza
  	 , runner_orders_fix.pickup_time::timestamp AS pickup_time
  	 , customer_orders_fix.order_time::timestamp AS order_time
  FROM customer_orders_fix
  LEFT JOIN runner_orders_fix
  ON runner_orders_fix.order_id = customer_orders_fix.order_id
  WHERE pickup_time IS NOT NULL
  GROUP BY customer_orders_fix.order_id, pickup_time, order_time
  ORDER BY customer_orders_fix.order_id, pickup_time, order_time
)

SELECT	DISTINCT num_pizza
	, ROUND(AVG(EXTRACT(EPOCH FROM pickup_time - order_time)/60)::numeric, 2) AS avg_minutes
FROM pizza_count
GROUP BY num_pizza
ORDER BY num_pizza;
````

#### Answer

| num_pizza | avg_minutes |
| --------- | ----------- |
| 1         | 12.36       |
| 2         | 18.38       |
| 3         | 29.28       |

---

From the resulting query, it takes an average of:
- 12.36 minutes for orders with 1 pizza.
- 18.38 minutes for orders with 2 pizzas.
- 29.28 minutes for orders with 3 pizzas.

There appears to be a positive relationship between the number of pizzas per order and the average minutes it took to finish the order. To be more specific, increasing pizzas from 1 to 2 took Pizza Runner roughly an extra 6 minutes, while increasing from 2 to 3 took roughly an extra 11 minutes, suggesting there may be a non-linear relationship between the two variables.

***

**4. What was the average distance travelled for each customer?**

````sql
WITH cust_distance AS (
  SELECT customer_id
  		 , distance::decimal
  FROM customer_orders_fix
  LEFT JOIN runner_orders_fix
  ON customer_orders_fix.order_id = runner_orders_fix.order_id
  WHERE distance IS NOT NULL
)

SELECT	customer_id
		, AVG(distance) AS avg_distance
FROM cust_distance
GROUP BY customer_id
ORDER BY customer_id;
````

#### Answer

| customer_id | avg_distance |
| ----------- | ------------ |
| 101         | 20.00        |
| 102         | 16.73        |
| 103         | 23.40        |
| 104         | 10.00        |
| 105         | 25.00        |

---

From the resulting query, observe the average distance for each customer id in kilometers:
- Average distance traveled for customer 101 is 20.00km.
- Average distance traveled for customer 102 is 16.73km.
- Average distance traveled for customer 103 is 23.40km.
- Average distance traveled for customer 104 is 10.00km.
- Average distance traveled for customer 105 is 25.00km.

***

**5. What was the difference between the longest and shortest delivery times for all orders?**

````sql
WITH times_delivery AS (
  SELECT duration::int
  FROM runner_orders_fix
  WHERE duration IS NOT NULL
)

SELECT MAX(duration) - MIN(duration) AS time_diff
FROM times_delivery;
````

#### Answer

| time_diff |
| --------- |
| 30        |

---

From the resulting query, the time difference between the longest and shortest delivery times for all non-null orders is 30 minutes.

***

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

````sql
WITH delivery_speed AS (
  SELECT runner_id
  		 , order_id
  		 , distance::numeric
 		 , duration::numeric
  FROM runner_orders_fix
  WHERE duration IS NOT NULL
)

SELECT
	runner_id
    , order_id
    , ROUND(AVG(distance), 2) AS delivery_distance
    , ROUND(AVG(duration / 60), 2) AS delivery_time
    , ROUND(SUM(distance) / (SUM(duration) / 60), 2) AS delivery_speed
FROM delivery_speed
GROUP BY runner_id, order_id
ORDER BY runner_id, order_id;
````

#### Answer

| runner_id | order_id | delivery_distance | delivery_time | delivery_speed |
| --------- | -------- | ----------------- | ------------- | -------------- |
| 1         | 1        | 20.00             | 0.53          | 37.50          |
| 1         | 2        | 20.00             | 0.45          | 44.44          |
| 1         | 3        | 13.40             | 0.33          | 40.20          |
| 1         | 10       | 10.00             | 0.17          | 60.00          |
| 2         | 4        | 23.40             | 0.67          | 35.10          |
| 2         | 7        | 25.00             | 0.42          | 60.00          |
| 2         | 8        | 23.40             | 0.25          | 93.60          |
| 3         | 5        | 10.00             | 0.25          | 40.00          |

---

From the resulting query, one can observe that in general, runner 2 was faster in making deliveries compared to runners 1 and 3. There are a few possible reasons as to why runner 2 is faster compared to the others, such as making deliveries by car vs. bike, familiarity with the neighborhood, etc.. 

***

**7. What is the successful delivery percentage for each runner?**

````sql
WITH successful_deliveries AS (
  SELECT runner_id
	 , CASE
	 WHEN cancellation IS NULL THEN 1
	 ELSE 0 END AS success
  FROM runner_orders_fix
)
,	calculations AS (
  SELECT runner_id
	 , SUM(success)::numeric AS deliveries_success
	 , COUNT(success)::numeric AS deliveries_total
  FROM successful_deliveries
  GROUP BY runner_id
  ORDER BY runner_id
)

SELECT
	runner_id
	, deliveries_success
	, deliveries_total
	, ROUND((deliveries_success/deliveries_total) * 100, 0) AS success_rate 
FROM calculations
ORDER BY runner_id;
````

#### Answer

| runner_id | deliveries_success | deliveries_total | success_rate |
| --------- | ------------------ | ---------------- | ------------ |
| 1         | 4                  | 4                | 100          |
| 2         | 3                  | 4                | 75           |
| 3         | 1                  | 2                | 50           |

---

From the resulting query, observe the following:
- Runner 1 successfully completed 100% of their orders.
- Runner 2 successfully completed 75% of their orders.
- Runner 3 successfully completed 50% of their orders.

***

### C. Ingredient Optimisation

**1. What are the standard ingredients for each pizza?**

````sql
WITH temp_ing AS (
  SELECT
  	pizza_id
  	, UNNEST(STRING_TO_ARRAY(toppings, ', '))::int AS n_toppings
  FROM pizza_runner.pizza_recipes
)

SELECT
	temp_ing.pizza_id
	, pizza_toppings.topping_name
FROM temp_ing
LEFT JOIN pizza_runner.pizza_toppings
ON temp_ing.n_toppings = pizza_toppings.topping_id
ORDER BY temp_ing.pizza_id, temp_ing.n_toppings;
````

#### Code Explanation
Used the new statements 'UNNEST' and 'STRING_TO_ARRAY' to 'explode' the numbers in the 'toppings' column in table pizza_recipes. UNNEST takes an array and returns a table with a row for each element in the array, while the STRING_TO_ARRAY splits the string in the original 'toppings' observation using the delimiter ','.

#### Answer

| pizza_id | topping_name |
| -------- | ------------ |
| 1        | Bacon        |
| 1        | BBQ Sauce    |
| 1        | Beef         |
| 1        | Cheese       |
| 1        | Chicken      |
| 1        | Mushrooms    |
| 1        | Pepperoni    |
| 1        | Salami       |
| 2        | Cheese       |
| 2        | Mushrooms    |
| 2        | Onions       |
| 2        | Peppers      |
| 2        | Tomatoes     |
| 2        | Tomato Sauce |

---

***

**2. What was the most commonly added extra?**

````sql
WITH temp_extras AS (
  SELECT
  	UNNEST(STRING_TO_ARRAY(extras, ', '))::int AS n_extras
  FROM customer_orders_fix
  WHERE extras IS NOT NULL
)

SELECT
	pizza_toppings.topping_name
    	, COUNT(n_extras) AS freq
FROM temp_extras
LEFT JOIN pizza_runner.pizza_toppings
ON n_extras = pizza_toppings.topping_id
GROUP BY pizza_toppings.topping_name
ORDER BY freq DESC
LIMIT 1;
````


#### Answer

| topping_name | freq |
| ------------ | ---- |
| Bacon        | 4    |

---

Bacon was the most common extra added, with 4 customer orders.

***

**3. What was the most common exclusion?**

````sql
WITH temp_exclu AS (
  SELECT
  	UNNEST(STRING_TO_ARRAY(exclusions, ', '))::int AS n_exclude
  FROM customer_orders_fix
  WHERE exclusions IS NOT NULL
)

SELECT
	pizza_toppings.topping_name
    , COUNT(n_exclude) AS freq
FROM temp_exclu
LEFT JOIN pizza_runner.pizza_toppings
ON n_exclude = pizza_toppings.topping_id
GROUP BY pizza_toppings.topping_name
ORDER BY freq DESC
LIMIT 1;
````


#### Answer

| topping_name | freq |
| ------------ | ---- |
| Cheese       | 4    |

---

Cheese was the most common ingredient excluded from a total of 4 customer orders.

***

**4. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients.**

````sql
SELECT 
	order_id
	, CASE 
    WHEN extras LIKE '1, 4' AND exclusions LIKE '2, 6'
    THEN CONCAT(pizza_names.pizza_name, ' - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Chicken')
    WHEN extras LIKE '1, 5' AND exclusions LIKE '4'
    THEN CONCAT(pizza_names.pizza_name, ' - Exclude Cheese - Extra Bacon, Chicken')
    WHEN exclusions IS NULL AND extras LIKE '1'
    THEN CONCAT(pizza_names.pizza_name, ' - Extra Bacon')
    WHEN extras IS NULL AND exclusions LIKE '4'
    THEN CONCAT(pizza_names.pizza_name, ' - Exclude Cheese')
    ELSE pizza_names.pizza_name END AS order_ticket
FROM customer_orders_fix
LEFT JOIN pizza_runner.pizza_names
ON customer_orders_fix.pizza_id = pizza_names.pizza_id
ORDER BY customer_orders_fix.order_id, customer_orders_fix.pizza_id;
````

#### Code Explanation
There wasn't a clean way to do this lookup without a large amount of data cleaning. Currently hard-coded all of the different combinations but will try to come up with a more elegant solution at a later time.

#### Answer

| order_id | order_ticket                                                     |
| -------- | ---------------------------------------------------------------- |
| 1        | Meatlovers                                                       |
| 2        | Meatlovers                                                       |
| 3        | Meatlovers                                                       |
| 3        | Vegetarian                                                       |
| 4        | Meatlovers - Exclude Cheese                                      |
| 4        | Meatlovers - Exclude Cheese                                      |
| 4        | Vegetarian - Exclude Cheese                                      |
| 5        | Meatlovers - Extra Bacon                                         |
| 6        | Vegetarian                                                       |
| 7        | Vegetarian - Extra Bacon                                         |
| 8        | Meatlovers                                                       |
| 9        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken               |
| 10       | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Chicken |
| 10       | Meatlovers                                                       |

---

***

**5. Generate an order item for each record in the customers_orders table in the format of one of the following:**

````sql
SELECT 
	order_id
	, CASE 
		WHEN customer_orders_fix.pizza_id = 1 AND extras LIKE '1, 4' AND exclusions LIKE '2, 6'
		THEN CONCAT(pizza_names.pizza_name, ': 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami')
		WHEN customer_orders_fix.pizza_id = 1 AND extras LIKE '1, 5' AND exclusions LIKE '4'
		THEN CONCAT(pizza_names.pizza_name, ': 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami')
		WHEN customer_orders_fix.pizza_id = 1 AND exclusions IS NULL AND extras LIKE '1'
		THEN CONCAT(pizza_names.pizza_name, ': 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami')
		WHEN customer_orders_fix.pizza_id = 2 AND exclusions IS NULL AND extras LIKE '1'
		THEN CONCAT(pizza_names.pizza_name, ': Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce')
		WHEN customer_orders_fix.pizza_id = 1 AND extras IS NULL AND exclusions LIKE '4'
		THEN CONCAT(pizza_names.pizza_name, ': 2x Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami')
		WHEN customer_orders_fix.pizza_id = 2 AND extras IS NULL AND exclusions LIKE '4'
		THEN CONCAT(pizza_names.pizza_name, ': Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce')
		WHEN customer_orders_fix.pizza_id = 1 AND extras IS NULL AND exclusions IS NULL
		THEN CONCAT(pizza_names.pizza_name, ': Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami')
		ELSE CONCAT(pizza_names.pizza_name, ': Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce') END AS order_ticket
FROM customer_orders_fix
LEFT JOIN pizza_runner.pizza_names
ON customer_orders_fix.pizza_id = pizza_names.pizza_id
ORDER BY customer_orders_fix.order_id, customer_orders_fix.pizza_id;
````

#### Code Explanation
There wasn't a clean way to do this lookup without a large amount of data cleaning. Currently hard-coded all of the different combinations but will try to come up with a more elegant solution at a later time.

#### Answer

| order_id | order_ticket                                                                         |
| -------- | ------------------------------------------------------------------------------------ |
| 1        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 2        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 4        | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami         |
| 4        | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami         |
| 4        | Vegetarian: Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                       |
| 5        | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 7        | Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce        |
| 8        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 9        | Meatlovers: 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami      |
| 10       | Meatlovers: 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami                    |
| 10       | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |

---

***

**6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**

````sql
WITH CTE AS (
  SELECT customer_orders_fix.order_id
  		 , customer_orders_fix.customer_id
  		 , customer_orders_fix.pizza_id
  		 , customer_orders_fix.exclusions
  		 , customer_orders_fix.extras
  		 , customer_orders_fix.order_time
  FROM customer_orders_fix
  LEFT JOIN runner_orders_fix
  ON customer_orders_fix.order_id = runner_orders_fix.order_id
  WHERE runner_orders_fix.cancellation IS NULL)


SELECT 
	order_id
	, CASE 
    WHEN CTE.pizza_id = 1 AND extras LIKE '1, 4' AND exclusions LIKE '2, 6'
    THEN CONCAT(pizza_names.pizza_name, ': 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami')
    WHEN CTE.pizza_id = 1 AND extras LIKE '1, 5' AND exclusions LIKE '4'
    THEN CONCAT(pizza_names.pizza_name, ': 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami')
    WHEN CTE.pizza_id = 1 AND exclusions IS NULL AND extras LIKE '1'
    THEN CONCAT(pizza_names.pizza_name, ': 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami')
    WHEN CTE.pizza_id = 2 AND exclusions IS NULL AND extras LIKE '1'
    THEN CONCAT(pizza_names.pizza_name, ': Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce')
    WHEN CTE.pizza_id = 1 AND extras IS NULL AND exclusions LIKE '4'
    THEN CONCAT(pizza_names.pizza_name, ': 2x Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami')
    WHEN CTE.pizza_id = 2 AND extras IS NULL AND exclusions LIKE '4'
    THEN CONCAT(pizza_names.pizza_name, ': Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce')
    WHEN CTE.pizza_id = 1 AND extras IS NULL AND exclusions IS NULL
    THEN CONCAT(pizza_names.pizza_name, ': Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami')
    ELSE CONCAT(pizza_names.pizza_name, ': Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce') END AS order_ticket
FROM CTE
LEFT JOIN pizza_runner.pizza_names
ON CTE.pizza_id = pizza_names.pizza_id
ORDER BY CTE.order_id, CTE.pizza_id;
````

#### Code Explanation
There wasn't a clean way to do this lookup without a large amount of data cleaning. Currently hard-coded all of the different combinations but will try to come up with a more elegant solution at a later time.

#### Answer

| order_id | order_ticket                                                                         |
| -------- | ------------------------------------------------------------------------------------ |
| 1        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 2        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 4        | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami         |
| 4        | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami         |
| 4        | Vegetarian: Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                       |
| 5        | Meatlovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 7        | Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce        |
| 8        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 10       | Meatlovers: 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami                    |
| 10       | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |

---
Bacon - 14
BBQ Sauce - 8
Beef - 9
Cheese - 10
Chicken - 9
Mushrooms - 11
Onions - 3
Pepperoni - 9
Peppers - 3
Salami - 9
Tomatoes - 3
Tomato Sauce - 3

From the query above, Bacon was ordered most frequently at 14 times. The next most frequently utilized ingredient was Mushrooms at 11 times, then the other ingredients in Meatlovers at mostly 9 times (BBQ Sauce was excluded once).

***

### D. Pricing and Ratings

**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

````sql
WITH cte AS (
  SELECT 
  	CASE WHEN c.pizza_id = 1 THEN 12
    ELSE 10 END AS revenue
  FROM customer_orders_fix c
  LEFT JOIN runner_orders_fix r
  ON c.order_id = r.order_id
  WHERE r.cancellation IS NULL
)

SELECT SUM(revenue) AS tot_revenue
FROM cte;
````

#### Code Explanation
May be simpler to SUM the CASE WHEN statements in the CTE.

#### Answer

| tot_revenue |
| ----------- |
| 138         |

---

Notice that total revenue for all successfully delivered orders was $138 if a Meat Lovers pizza costs $12 and a Vegetarian costs $10.  

***

**2. What if there was an additional $1 charge for any pizza extras?**

````sql
WITH cte AS (
	SELECT
		*
		, UNNEST(STRING_TO_ARRAY(extras, ', '))::int AS n_extras
	FROM customer_orders_fix
	WHERE extras IS NOT NULL
)
, cte2 AS (
	SELECT
		order_id
		, COUNT(n_extras) AS n_extras
	FROM cte
	GROUP BY order_id)
, cte_customer AS (
	SELECT
		c.order_id
		, SUM(CASE WHEN 
			c.pizza_id = 1 THEN 12 ELSE 10 END) AS revenue
	FROM customer_orders_fix c
	LEFT JOIN runner_orders_fix r
	ON c.order_id = r.order_id
	WHERE r.cancellation IS NULL
	GROUP BY c.order_id
	ORDER BY c.order_id
)

SELECT SUM(CASE WHEN b.n_extras is NULL THEN a.revenue
	ELSE a.revenue + b.n_extras END) AS total_revenue
FROM cte_customer a
LEFT JOIN cte2 b
ON a.order_id = b.order_id;
````

#### Code Explanation
Probably not the most elegant solution, but this code executes in the following blocks:
- Split out extras in 'extra' column of the customer_orders_fix table in initial CTE.
- Estimate the total revenue from additional extras on pizza orders in CTE 2, where extras are priced at $1 per extra.
- Estimate total revenue from actual pizza orders that are successfully delivered where Meatlover pizzas are priced at $12 and Vegetarian pizzas are priced at $10.

#### Answer

| total_revenue |
| ------------- |
| 142           |

---

***

**3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**

````sql
DROP TABLE IF EXISTS runner_ratings;
CREATE TABLE runner_ratings (
  "order_id" INTEGER,
  "rating" INTEGER
);
INSERT INTO runner_ratings
  ("order_id", "rating")
VALUES
  (1, 4),
  (2, 3),
  (3, 2),
  (5, 3),
  (7, 5),
  (8, 1),
  (10, 3);
````

#### Code Explanation
The schema is extremely simple and does not require too much information, as one can join the other available schemas that contain the information one needs to conduct various analyses. Looking just at the ratings, one can look at the range of ratings that each customer had for runners responsible for their delivery, since a single runner is responsible for a single order. Potentially, there could be additional information that is interesting to ensure data quality is reliable, such as the following:
- Review time, which records the time a customer rated their experience after delivery.
- Comments, where customers can record what went wrong with their delivery. This is to potentially filter out customer ratings based on the food, rather than the runner delivery itself.

***

**4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?**
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

#### Code Explanation
Using the schema designed in Question 3, it would be possible to come up with a table that looks like the above. However, it does not seem as if the information provided in the new table would be very useful outside of providing a loose overview of the available data. The only aggregation that looks particularly useful is the 'Total number of pizzas' column, which would provide a total number of pizzas that were ordered under each order_id. On the other hand, if the purpose of this combined file is to create an overall database that contains updated information of overall averages of each delivery, that could be helpful. However, an issue with this approach is that it is unclear which id should serve as the base for the analysis. At a glance, runner_id appears to be the most appropriate base for average speed, where one can join a "speed by runner" variable from a CTE. An issue with this approach would be that there would be duplicates throughout the column.

***

**5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**

````sql
WITH cte AS (
  SELECT
  	c.order_id
  	, SUM(CASE WHEN
  		c.pizza_id = 1 THEN 12
  		ELSE 10 END) AS pizza_revenue
  FROM customer_orders_fix c
  LEFT JOIN runner_orders_fix r
  ON c.order_id = r.order_id
  WHERE r.cancellation IS NULL
  GROUP BY c.order_id
), cte2 AS (
  SELECT
  	r.order_id
  	, SUM(CASE WHEN 
  		r.distance IS NULL THEN 0
  		ELSE r.distance::numeric * 0.3 END) AS runner_expense
  FROM runner_orders_fix r
  WHERE r.cancellation IS NULL
  GROUP BY r.order_id
)

SELECT
	SUM(a.pizza_revenue - b.runner_expense) AS total_profit
FROM cte a
LEFT JOIN cte2 b
ON a.order_id = b.order_id;
````

#### Answer

| total_profit |
| ------------ |
| 94.44        |

***

### E. Bonus Questions

** If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?**

- Use the following syntax for an 'INSERT' statement in SQL:
````sql
INSERT INTO pizza_recipes("pizza_id", "toppings");
VALUES(3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');
````
