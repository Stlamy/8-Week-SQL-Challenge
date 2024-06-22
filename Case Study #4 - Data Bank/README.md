	# Case Study #3 - Foodie-Fi
<image src = "https://8weeksqlchallenge.com/images/case-study-designs/4.png" alt = "Image" width = "500" height = "520">

## Index
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Solutions](#solutions)
	- [A. Customer Nodes Exploration](#customer-nodes-exploration)
  - [B. Customer Transactions](#customer-transactions)
  - [C. Data Allocation Challenge](#data-allocation-challenge)
  - [D. Extra Challenge](#extra-challenge)

***

Thanks to Danny for sharing this challenge. You can find the questions and schema used in [this link](https://8weeksqlchallenge.com/case-study-4/).

## Business Task
A new innovation in the financial industry is called 'Neo-Banks', or digital-only banks without physical branches. Danny thought there should be some sort of intersection between these new banks and the data world, so he launched Data Bank, which not only offers banking activities but also provides the world's most secure distributed data storage platform. Customers are allocated cloud data storage limits directly linked to the amount held in their bank accounts.

Data Bank wants to increase their total customer base but also need help tracking how much data storage their customers will need.

***

## Entity Relationship Diagram
<image src = "https://user-images.githubusercontent.com/81607668/129744449-37b3229b-80b2-4cce-b8e0-707d7f48dcec.png" alt = "Image" width = "500" height = "260">

Data is stored off of a network of 'nodes' where both money and data is stored. There are 5 regions for some of the major continents of the world:
1. Africa
2. America
3. Asia
4. Europe
5. Oceania

The second table contains customer nodes, where each customer is randomly distributed across the nodes according to their region. It is not clear what the start and end dates are, since the differences in dates seem quite small for bank accounts, which are generally held for a longer period of time. 

The last table stores all customer deposits, withdrawals, and purchases made using ther Data Bank debit card. It is also unclear what currency the customers are using.

***

## Solutions
Execute the queries using PostgreSQL on DB Fiddle. 

***

## Case Study Questions

### A. Customer Nodes Exploration

**1. How many unique nodes are there on the Data Bank system?**

````sql
SELECT COUNT(DISTINCT node_id) AS node_count
FROM data_bank.customer_nodes;
````

| node_count |
| ---------- |
| 5          |

---

#### Answer
There are 5 unique customer nodes in the Data Bank system.

---

***

**2. What is the number of nodes per region?**

````sql
SELECT
    a.region_id
    , b.region_name
    , COUNT(DISTINCT a.node_id) AS unique_count
    , COUNT(a.node_id) AS total_count
FROM data_bank.customer_nodes a
LEFT JOIN data_bank.regions b
ON a.region_id = b.region_id
GROUP BY a.region_id, b.region_name
ORDER BY a.region_id, b.region_name;
````

| region_id | region_name | unique_count | total_count |
| --------- | ----------- | ------------ | ----------- |
| 1         | Australia   | 5            | 770         |
| 2         | America     | 5            | 735         |
| 3         | Africa      | 5            | 714         |
| 4         | Asia        | 5            | 665         |
| 5         | Europe      | 5            | 616         | 

---

#### Answer
The question did not specify whether they wanted the count of unique or total nodes by each region, so ran both counts. From the query above, total count of nodes in each region ranges from 616-770, where Australia has the most and Europe the least. However, each region has 5 unique nodes each.

---

***

**3. How many customers are allocated to each region?**

````sql
SELECT
    a.region_id
    , b.region_name
    , COUNT(DISTINCT a.customer_id) AS unique_count
    , COUNT(a.customer_id) AS total_count
FROM data_bank.customer_nodes a
LEFT JOIN data_bank.regions b
ON a.region_id = b.region_id
GROUP BY a.region_id, b.region_name
ORDER BY a.region_id, b.region_name;
````

| region_id | region_name | unique_count | total_count |
| --------- | ----------- | ------------ | ----------- |
| 1         | Australia   | 110          | 770         |
| 2         | America     | 105          | 735         |
| 3         | Africa      | 102          | 714         |
| 4         | Asia        | 95           | 665         |
| 5         | Europe      | 88           | 616         |

---

#### Answer
Looking at the query above, total and unique customer counts seem to follow the pattern of total nodes above in question 3.

---

***

**4. How many days on average are customers reallocated to a different node?**

````sql
SELECT 
	ROUND(AVG(end_date - start_date), 0) AS avg_days
FROM data_bank.customer_nodes;
````


#### Answer

| avg_days |
| -------- |
| 416373   |

---

From the query above, there seems to be a data issue, as it appears that it is taking customers an average of 416,373 days to reallocate to a different node. To correct this, look for the row with the data issue.

````sql
WITH cte AS (
  SELECT
  	*
  	, EXTRACT(YEAR FROM start_date) AS start_year
  	, EXTRACT(YEAR FROM end_date) AS end_year
  FROM data_bank.customer_nodes
)

SELECT 
	start_year
    , MAX(start_year) AS max_start
    , end_year
    , MAX(end_year) AS max_end
FROM cte
GROUP BY start_year, end_year
ORDER BY start_year, end_year;
````

| start_year | end_year  |
| ---------- | --------- |
| 2020       | 2020      |
| 2020       | 9999      |

---

From the query above, observe a max end_year of 9999, which is likely to be a data issue. Correct for this issue by removing these rows when estimating the average days customers are reallocated to a new node.

````sql
WITH cte AS (
  SELECT
  	*
  	, EXTRACT(YEAR FROM start_date) AS start_year
  	, EXTRACT(YEAR FROM end_date) AS end_year
  FROM data_bank.customer_nodes
)

SELECT 
	ROUND(AVG(end_date - start_date), 0) AS avg_days
FROM cte
WHERE end_year != 9999;
````

| avg_days |
| -------- |
| 15       |

Observe a corrected estimate of an average of 15 days to reallocate customers to a different node.

***

**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**

````sql
WITH cte AS (
  SELECT
  	*
  	, end_date - start_date AS day_diff
  FROM data_bank.customer_nodes
  WHERE EXTRACT(YEAR FROM end_date) != 9999
)

SELECT 
	PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY day_diff) AS "Median"
  	, PERCENTILE_CONT(0.8) WITHIN GROUP(ORDER BY day_diff) AS "80th Percentile"
  	, PERCENTILE_CONT(0.95) WITHIN GROUP(ORDER BY day_diff) AS "95th Percentile"
FROM cte;
````

#### Code Explanation
- Notice the new PERCENTILE_CONT statement, which estimates the percentile for a continuous variable, which in this case represents the difference between the start and end dates.

| Median | 80th Percentile | 95th Percentile |
| ------ | --------------- | --------------- |
| 15     | 23              | 28              |

---

#### Answer
From the query above, observe the following reallocation metrics for the following percentiles:
- Median (50%) = 15 days
- 80th Percentile = 23 days
- 95th Percentile = 28 days

---

***

### B. Customer Transactions

**1. What is the unique count and total amount for each transaction type?**

````sql
SELECT
	txn_type
	, COUNT(txn_type) AS txn_count
	, SUM(txn_amount) AS txn_total
FROM data_bank.customer_transactions
GROUP BY txn_type
ORDER BY txn_total DESC;
````

| txn_type   | txn_count | txn_total |
| ---------- | --------- | --------- |
| deposit    | 2671      | 1359168   |
| purchase   | 1617      | 806537    |
| withdrawal | 1580      | 793003    |

---

#### Answer
The most frequent transaction type is deposits at 2,671 total transactions and 1,359,168 in total transaction amount. The least frequent transaction is withdrawals at 1,580 total transactions, with 793,003 in total transaction amount. Purchases are in between, but much closer to withdrawals compared to deposits, which means customers tend to hold their finances in Data Bank rather than use it as a spending account. 

***

**2. What is the average total historical deposit counts and amounts for all customers?**

````sql
WITH cte AS (
  SELECT
  	customer_id
  	, COUNT(txn_type) AS dep_count
  	, AVG(txn_amount) AS avg_deposit
  FROM data_bank.customer_transactions
  WHERE txn_type LIKE 'deposit'
  GROUP BY customer_id
  ORDER BY customer_id
)

SELECT
	ROUND(AVG(dep_count), 0) AS dep_count
	, ROUND(AVG(avg_deposit), 2) AS avg_deposit
FROM cte;
````

| dep_count  | avg_deposit |
| ---------- | ----------- |
| 5          | 508.61      |

---

#### Answer
Data Bank customers, on average, made 5 deposits of 508.61 for each deposit transaction.

***

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**

````sql
WITH cte1 AS (
  SELECT
  	*
  	, EXTRACT(MONTH FROM txn_date) AS txn_month
  	, CASE WHEN txn_type LIKE 'deposit' THEN 1 ELSE 0 END AS deposit_flag
  	, CASE WHEN txn_type NOT LIKE 'deposit' THEN 1 ELSE 0 END AS txn_flag
  FROM data_bank.customer_transactions
), cte AS (
  SELECT
  	customer_id
  	, txn_month
  	, SUM(deposit_flag) AS total_deposit
  	, SUM(txn_flag) AS other_txn
  FROM cte1
  GROUP BY customer_id, txn_month
  ORDER BY customer_id, txn_month
) 

SELECT
	txn_month
	, COUNT(DISTINCT customer_id) AS customer_count
FROM cte
WHERE total_deposit > 1 AND other_txn = 1
GROUP BY txn_month
ORDER BY txn_month;
````

| txn_month | customer_count |
| --------- | -------------- |
| 1         | 53             |
| 2         | 36             |
| 3         | 38             |
| 4         | 22             |

---

#### Answer
From the query above:
- In January, there were 53 customers with more than 1 deposit and exactly 1 purchase or withdrawal.
- In February, there were 36 customers with more than 1 deposit and exactly 1 purchase or withdrawal.
- In March, there were 38 customers with more than 1 deposit and exactly 1 purchase or withdrawal.
- In April, there were 22 customers with more than 1 deposit and exactly 1 purchase or withdrawal.

***

**4. What is the closing balance for each customer at the end of the month?**

````sql
WITH cte1 AS (
  SELECT
  	*
  	, EXTRACT(YEAR FROM txn_date) AS txn_year
  	, EXTRACT(MONTH FROM txn_date) AS txn_month
  	, CASE WHEN
  		txn_type LIKE 'deposit' THEN txn_amount
  		ELSE txn_amount * (-1) END AS balance
  FROM data_bank.customer_transactions
), cte2 AS (
  SELECT
	customer_id
    , txn_year
    , txn_month
    , SUM(balance) AS initial_balance
  FROM cte1
  GROUP BY customer_id, txn_year, txn_month
  ORDER BY customer_id, txn_year, txn_month
), cte AS (
  SELECT
  	customer_id
  	, txn_year
  	, txn_month
  	, initial_balance
  	, LAG(initial_balance, 1) OVER (PARTITION BY customer_id ORDER BY customer_id) AS balance_adjustment1
  	, LAG(initial_balance, 2) OVER (PARTITION BY customer_id ORDER BY customer_id) AS balance_adjustment2
  	, LAG(initial_balance, 3) OVER (PARTITION BY customer_id ORDER BY customer_id) AS balance_adjustment3
  FROM cte2
)

SELECT
	customer_id
  	, txn_year
  	, txn_month
  	, (initial_balance
       + COALESCE(balance_adjustment1, 0)
       + COALESCE(balance_adjustment2, 0)
       + COALESCE(balance_adjustment3, 0)) AS final_balance
FROM cte;
````

- Note: Query output has been omitted for brevity.

#### Code Explanation
This is most likely not the most elegant solution to this question, as balance adjustments are made manually rather than using SQL statements. 
Notice the LAG statement in the cte section, which pulls the previous balance where available across customer_id and transaction month. This allows operations between the previous month's balance and the current month's balance to estimate changes in the actual month-end balance. However, since that only brings in the previous month's balance, previous month adjustments are brought into each row using different offset values, since there are only 4 months included in this dataset. By adding the row values across all 4 columns (using the COALESCE statement to ensure that 'null' values are counted as 0 to avoid null output), the query outputs the monthly balance based on previous month observations where available. 

***

**5. What is the percentage of customers who increase their closing balance by more than 5%?**

````sql
WITH cte1 AS (
  SELECT
  	*
  	, EXTRACT(YEAR FROM txn_date) AS txn_year
  	, EXTRACT(MONTH FROM txn_date) AS txn_month
  	, CASE WHEN
  		txn_type LIKE 'deposit' THEN txn_amount
  		ELSE txn_amount * (-1) END AS balance
  FROM data_bank.customer_transactions
), cte2 AS (
  SELECT
	customer_id
    , txn_year
    , txn_month
    , SUM(balance) AS initial_balance
  FROM cte1
  GROUP BY customer_id, txn_year, txn_month
  ORDER BY customer_id, txn_year, txn_month
), cte3 AS (
  SELECT
  	customer_id
  	, txn_year
  	, txn_month
  	, initial_balance
  	, LAG(initial_balance, 1) OVER (PARTITION BY customer_id ORDER BY customer_id) AS balance_adjustment1
  	, LAG(initial_balance, 2) OVER (PARTITION BY customer_id ORDER BY customer_id) AS balance_adjustment2
  	, LAG(initial_balance, 3) OVER (PARTITION BY customer_id ORDER BY customer_id) AS balance_adjustment3
  FROM cte2
), cte AS (
  SELECT
	customer_id
  	, txn_year
  	, txn_month
  	, (initial_balance
       + COALESCE(balance_adjustment1, 0)
       + COALESCE(balance_adjustment2, 0)
       + COALESCE(balance_adjustment3, 0)) AS final_balance
  	, DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY txn_month DESC) AS balance_order
  FROM cte3
), cte_initial1 AS (
  SELECT
  	*
  	, DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY txn_date) AS first_deposit
  FROM data_bank.customer_transactions
  WHERE txn_type LIKE 'deposit'
), cte_initial AS (
  SELECT
  	*
  FROM cte_initial1
  WHERE first_deposit = 1
), cte_final AS (
  SELECT
  	(final_balance - txn_amount)*100.0/txn_amount AS balance_increase
  FROM cte a
  LEFT JOIN cte_initial b
  ON a.customer_id = b.customer_id
  WHERE a.balance_order = 1
)

SELECT
	COUNT(balance_increase) AS balance_count
FROM cte_final
WHERE balance_increase >= 5;
````

#### Answer
This question is very confusing as it does not define the 'time period' to compare balances to determine an increase in customers' closing balances. In theory, the query that would use to estimate this question would require either the LAG() statement or the DENSE_RANK() statement in order to compare balances between sequential months. Assuming the question is asking for an increase in closing balance from when they first open their account and the latest month available. Again, this is not the most elegant solution, but there are two CTE processes; the first is identical to question 4, while the second extracts the initial deposit for each customer. In the final CTE, these two parts are combined into a single table and estimates the balance increase rate. Finally, count the number of customers with balance increases greater than 5 from their initial deposit to their last.

***

### C. Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:
- Option 1: data is allocated based off the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real-time

For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:
- running customer balance column that includes the impact each transaction
- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer

Using all of the data available - how much data would have been required for each option on a monthly basis?

** **

Plan to update at a later date.

***

### D. Extra Challenge

**1. blank**

***
