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

````


#### Answer

---

***

### B. Customer Transactions

**1. blank**

***

### C. Data Allocation Challenge

**1. blank**

***

### D. Extra Challenge

**1. blank**

***
