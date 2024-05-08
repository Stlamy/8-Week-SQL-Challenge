# Case Study #1 - Danny's Diner
<image src = "https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt = "Image" width = "500" height = "520">

## Index
- Business Task
- Entity Relationship Diagram
- Solutions

***

## Business Task
Danny has a restaurant that sells his three favorite foods: sushi, curry, and ramen. He wants to know his customer's visiting patterns, money spent, and favourite menu items in order to decide whether he should expand the existing customer loyalty program. Danny has provided us with the following three tables to answer his questions.

***

## Entity Relationship Diagram


***

## Solutions
Execute the queries using PostgreSQL on DB Fiddle. 

***

### Case Study Questions
**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT
    sales.customer_id,
    SUM(menu.price) AS tot_sales
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.tot_sales;
````

#### Code Explanation
- Use the SELECT statement to define variables to pull from a database.
- Use the SUM statement to aggregate observations by the specified variable.
- Use the LEFT JOIN statement to append matching records from a referenced second table based on a matching key between the two tables.

#### Answer
| customer_id | tot_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

From the resulting query, observe the following sales:
- Customer A spent 76
- Customer B spent 74
- Customer C spent 36

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT
  	sub_q.customer_id
    , COUNT(sub_q.customer_id) AS tot_visits
FROM (SELECT
      	sales.customer_id
      	, sales.order_date
      FROM dannys_diner.sales
      GROUP BY sales.customer_id
      		   , sales.order_date) AS sub_q
GROUP BY sub_q.customer_id
ORDER BY tot_visits DESC;
````

#### Code Explanation
- Use the COUNT statement to generate a count of matching observations by the specified variable.
- Use a subquery defined as sub_q to count the number of unique visits by order date for each customer_id.
- Ordered the customer_id by most visits with the DESC statement.

#### Answer
| customer_id | tot_visits |
| ----------- | ----------- |
| B           | 6          |
| A           | 4          |
| C           | 2          |

From the resulting query, observe the following number of visits by each customer:
- Customer A visited 4 times
- Customer B visited 6 times
- Customer C visited 2 times

Note: There is a simpler way of using COUNT(DISTINCT var_name) to determine the number of unique order dates for each customer_id.

***

**3. What was the first item from the menu purchased by each customer?**

````sql
SELECT
	sub_q.customer_id
	, menu.product_name
FROM (SELECT 
      sales.customer_id
      , sales.product_id
      , DENSE_RANK() OVER (ORDER BY sales.order_date) AS first_order
      FROM dannys_diner.sales) AS sub_q
LEFT JOIN dannys_diner.menu
ON sub_q.product_id = menu.product_id
WHERE first_order = 1
GROUP BY sub_q.customer_id, menu.product_name
ORDER BY sub_q.customer_id;
````

#### Code Explanation
- Use DENSE_RANK() statement in order to create a 'rank' value for the specified variable (order_date), which starts from the defined order.
	- [Referenced here](https://www.sqltutorial.org/sql-window-functions/sql-dense_rank/)
 - Using a subquery like this example is less efficient and is convoluted compared to using a CTE (Common Table Expression).
 

#### Answer
| customer_id |  product_name |
| ----------- |  ------------ |
| A           |  curry        |
| A           |  sushi        |
| B           |  curry        |
| C           |  ramen        |

From the resulting query, the first item purchased by each customer was:
- Customer A purchased curry and sushi
- Customer B purchased curry
- Customer C purchased ramen

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT
  	menu.product_name,
	COUNT(sales.product_id) AS purchase_freq
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY purchase_freq DESC
LIMIT 1;
````

#### Answer
| product_name |  purchase_freq |
| ------------ |  ------------- |
| ramen        |  8             |

From the resulting query, the most frequently purchased item on Danny's menu was ramen at 8 times.

***

**5. Which item was the most popular for each customer?**

````sql
WITH
	order_count AS (SELECT sales.customer_id
			, sales.product_id
			, DENSE_RANK() OVER (PARTITION BY sales.customer_id
					     ORDER BY COUNT(sales.product_id) DESC) AS freq_rank
	FROM dannys_diner.sales
	GROUP BY sales.customer_id, sales.product_id)

SELECT
	order_count.customer_id
  	, menu.product_name
FROM order_count
LEFT JOIN dannys_diner.menu
ON order_count.product_id = menu.product_id
WHERE order_count.freq_rank = 1
GROUP BY order_count.customer_id, menu.product_name;
````

#### Code Explanation
- New option in DENSE_RANK() statement PARTITION BY, which lets SQL know that ranks should be done within group rather than across the entire table.
- Used a Common Table Expression (CTE) using the WITH statement, which is more efficient compared to using subqueries.

#### Answer
| customer_id |  product_name |
| ----------- |  ------------ |
| A	      |  ramen	      |
| B	      |  curry	      |
| B	      |  ramen	      |
| B	      |  sushi	      |
| C	      |  ramen	      |

From the resulting query, each customer purchased the following items most frequently:
- Customers A and C purchased ramen most frequently
- Customer B purchased all items equally

***

**6. Which item was purchased first by the customer after they became a member?**

Note: There is some uncertainty on whether customers become a member before or after the purchase. The following query assumes that customers are able to become members after ordering if order_date equals join_date.  

````sql
WITH
	first_p AS 
	(SELECT	members.*
     		, sales.order_date
     		, sales.product_id
     		, DENSE_RANK() OVER (PARTITION BY members.customer_id
				     ORDER BY sales.order_DATE) AS first_order
	FROM dannys_diner.members
	LEFT JOIN dannys_diner.sales
	ON members.customer_id = sales.customer_id
	WHERE sales.order_date > members.join_date
	ORDER BY members.customer_id, sales.order_date DESC)

SELECT	customer_id
        , menu.product_name
FROM first_p
LEFT JOIN dannys_diner.menu
ON first_p.product_id = menu.product_id
WHERE first_order = 1
ORDER BY customer_id;
````

#### Code Explanation
- WHERE statement defines which observations to filter observations that have an order_date before becoming a member. Particulary, we filter out observations to only those where order_date is greater than their member join_date.
 

#### Answer
| customer_id |  product_name |
| ----------- |  ------------ |
| A           |  ramen        |
| B           |  sushi        |

From the resulting query, the first item purchased by each customer after becoming a member was:
- Customer A's first purchase after becoming a member was 'ramen'.
- Customer B's first purchase after becoming a member was 'sushi'.

***

**7. Which item was purchased just before the customer became a member?**

Note: In continuity with problem 6, we assume that orders are made before the customer becomes a member when order_date = join_date.  

````sql
WITH
	first_p AS 
	(SELECT	members.*
     		, sales.order_date
     		, sales.product_id
     		, DENSE_RANK() OVER (PARTITION BY members.customer_id
				     ORDER BY sales.order_DATE DESC) AS first_order
	FROM dannys_diner.members
	LEFT JOIN dannys_diner.sales
	ON members.customer_id = sales.customer_id
	WHERE sales.order_date <= members.join_date
	ORDER BY members.customer_id, sales.order_date DESC)

SELECT	customer_id
        , menu.product_name
FROM first_p
LEFT JOIN dannys_diner.menu
ON first_p.product_id = menu.product_id
WHERE first_order = 1
ORDER BY customer_id;
````

#### Code Explanation
- WHERE statement defines which observations to filter observations that have an order_date after becoming a member. Particulary, we filter out observations to only those where order_date is less than or equal to their member join_date.
 

#### Answer
| customer_id |  product_name |
| ----------- |  ------------ |
| A           |  curry        |
| B           |  sushi        |

From the resulting query, the item purchased by each customer just before becoming a member was:
- Customer A: 'curry'.
- Customer B: 'sushi'.

***

**8. What is the total items and amount spent for each member before they became a member?**

````sql
WITH
	before_mem AS 
	(SELECT	sales.*
	FROM dannys_diner.sales
	LEFT JOIN dannys_diner.members
	ON sales.customer_id = members.customer_id
	WHERE sales.order_date <= members.join_date OR
	      members.join_date IS NULL
	ORDER BY members.customer_id)

SELECT	customer_id
        , COUNT(menu.price) AS tot_count
        , SUM(menu.price) AS tot_spend
FROM before_mem
LEFT JOIN dannys_diner.menu
ON before_mem.product_id = menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
````

#### Code Explanation
- There is an OR statement in the WHERE condition when constructing the CTE, which keeps all observations for customer C, since all of their observations are being dropped with NULL join_date.
 

#### Answer

| customer_id | tot_count | tot_spend |
| ----------- | --------- | --------- |
| A           | 3         | 40        |
| B           | 3         | 40        |
| C           | 3         | 36        |

From the resulting query, see the number and total spend by customer:
- Customer A spent $40 on 3 items.
- Customer B spent $40 on 3 items.
- Customer C spent $36 on 3 items.

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

````sql
WITH
	dol_spent AS 
	(SELECT	sales.customer_id
     		, menu.price
     		, CASE 
     			WHEN sales.product_id = 1 THEN 2 * 10 * menu.price
     			ELSE 10 * menu.price
            	END AS tot_points
	FROM dannys_diner.sales
	LEFT JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id)

SELECT	customer_id
        , SUM(tot_points) AS points_earned
FROM dol_spent
GROUP BY customer_id
ORDER BY customer_id;
````

#### Code Explanation
- Use the CASE statement with WHEN to create new variable "tot_points" based on the contents of sales.product_id and menu.price since "sushi" earns x2 points per dollar spent, while "ramen" and "curry" earn only x1. This variable is then summed by customer_id to estimate the total number of points that each customer could have earned if they were a member.
 

#### Answer

| customer_id | points_earned |
| ----------- | ------------- |
| A           | 860           |
| B           | 940           |
| C           | 360           |

From the resulting query, see the total points each customer could have earned if they were members at Danny's Diner:
- Customer A could have earned 860 points.
- Customer B could have earned 940 points.
- Customer C could have earned 360 points.

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

Note: Does not assume that customers receive 

````sql
WITH
	dol_spent AS 
	(SELECT	sales.customer_id
     		, menu.price
     		, CASE 
     			WHEN sales.product_id = 1 
     				OR sales.order_date < members.join_date + INTERVAL '7 DAY' THEN 2 * 10 * menu.price
     			ELSE 10 * menu.price
            	END AS tot_points
	FROM dannys_diner.sales
	LEFT JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
    LEFT JOIN dannys_diner.members
	ON sales.customer_id = members.customer_id
    WHERE sales.order_date < DATE('2021-02-01') 
     		AND members.join_date IS NOT NULL
    		AND sales.order_date > members.join_date)

SELECT	customer_id
        , SUM(tot_points) AS points_earned
FROM dol_spent
GROUP BY customer_id
ORDER BY customer_id;
````

#### Code Explanation
- Now including an OR statement to determine whether an order was made within a week of the join_date to earn x2 points on all items, not just "sushi". Observations without a join_date, order_date after '2021-02-01', or order_date on or after join_date were excluded. The last variable is consistent with prior answers that exclude orders made on the same day as join_date.
 

#### Answer

| customer_id | points_earned |
| ----------- | ------------- |
| A           | 720           |
| B           | 320           |

From the resulting query, see the total points each customer earned as a member at Danny's Diner:
- Customer A earned a total of 720 points.
- Customer B earned a total of 320 points.

***

### Bonus Questions
** Join All the Things **

#### Question: Recreate a table with the following columns: customer_id (string), order_date (date), product_name (string), price (integer), and member (boolean).
````sql
SELECT	sales.customer_id
	, DATE_TRUNC('day', sales.order_date)::date AS order_date
	, menu.product_name
	, menu.price
	, CASE 
		WHEN sales.order_date >= members.join_date AND members.join_date IS NOT NULL THEN 'Y'
		ELSE 'N'
            	END AS member
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
LEFT JOIN dannys_diner.members
ON sales.customer_id = members.customer_id
ORDER BY customer_id, order_date;
````

#### Code Explanation
- Now including an OR statement to determine whether an order was made within a week of the join_date to earn x2 points on all items, not just "sushi". Observations without a join_date, order_date after '2021-02-01', or order_date on or after join_date were excluded. The last variable is consistent with prior answers that exclude orders made on the same day as join_date.
 

#### Answer

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

***

