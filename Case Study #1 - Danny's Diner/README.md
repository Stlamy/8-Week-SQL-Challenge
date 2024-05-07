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

From the resulting table, observe the following sales:
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

From the resulting table, observe the following number of visits by each customer:
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
- Use DENSE_RANK() statement.
	- [Referenced here](https://www.sqltutorial.org/sql-window-functions/sql-dense_rank/)
 - 

#### Answer
| customer_id |  product_name |
| ----------- |  ------------ |
| A           |  curry        |
| A           |  sushi        |
| B           |  curry        |
| C           |  ramen        |

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
**5. Which item was the most popular for each customer?**
**6. Which item was purchased first by the customer after they became a member?**
**7. Which item was purchased just before the customer became a member?**
**8. What is the total items and amount spent for each member before they became a member?**
**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

### Bonus Questions
