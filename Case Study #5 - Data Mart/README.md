	# Case Study #3 - Foodie-Fi
<image src = "https://8weeksqlchallenge.com/images/case-study-designs/5.png" alt = "Image" width = "500" height = "520">

## Index
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Solutions](#solutions)
	- [1. Data Cleansing Steps](#data-cleansing-steps)
  - [2. Data Exploration](#data-exploration)
  - [3. Before & After Analysis](#before-after-analysis)
  - [4. Bonus Question](#bonus-question)

***

Thanks to Danny for sharing this challenge. You can find the questions and schema used in [this link](https://8weeksqlchallenge.com/case-study-5/).

## Business Task
Data Mart is Danny's latest venture, and he is asking for support to analyse his sales performance after running international operations for his online supermarket that specialises in fresh produce.

Danny made large scale supply changes to Data Mart in June 2020, where all products now use sustainable packaging methods every single step from farm to consumer. He needs help quantifying the impact of this change on sales performance for Data Mart and it's separate business areas.

He wants to know the following:
- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment and customer types were the most impacted by this change?
- What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

***

## Entity Relationship Diagram
<image src = "https://user-images.githubusercontent.com/81607668/129744449-37b3229b-80b2-4cce-b8e0-707d7f48dcec.png" alt = "Image" width = "500" height = "260">

There is only one table for this case study. Some further details:
1. Data Mart has international operations using a multi-region strategy
2. Data Mart has both, a retail and online platform in the form of a Shopify store front to serve their customers
3. Customer segment and customer_type data relates to personal age and demographics information that is shared with Data Mart
4. transactions is the count of unique purchases made through Data Mart and sales is the actual dollar amount of purchases

Each record is related to a specific aggregated slice of the underlying sales data rolled up into a week_date value which represents the start of the sales week.

***

## Solutions
Execute the queries using PostgreSQL on DB Fiddle. 

***

## Case Study Questions

### 1. Data Cleansing Steps

In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:
- Convert the week_date to a DATE format
- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a month_number with the calendar month for each week_date value as the 3rd column
- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

| segment | age_band     |
| ------- | ------------ | 
| 1       | Young Adults |
| 2       | Middle Aged  |
| 3 or 4  | Retirees     |

---

- Add a new demographic column using the following mapping for the first letter in the segment values:

| segment | demographic  |
| ------- | ------------ | 
| C       | Couples      |
| F       | Families     |

---

- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record


````sql
CREATE TEMP TABLE clean_weekly_sales AS
	SELECT
		TO_DATE(week_date, 'DD/MM/YY') AS week_date
		, EXTRACT(WEEK FROM TO_DATE(week_date, 'DD/MM/YY')) AS week_number
        	, EXTRACT(MONTH FROM TO_DATE(week_date, 'DD/MM/YY')) AS month_number
        	, EXTRACT(YEAR FROM TO_DATE(week_date, 'DD/MM/YY')) AS year_number
        	, region
        	, platform
        	, CASE WHEN segment LIKE 'null' THEN 'unknown' ELSE segment END AS segment
        	, CASE WHEN
        		segment LIKE '%1' THEN 'Young Adults'
          	WHEN
          		segment LIKE '%2' THEN 'Middle Aged'
          	ELSE 'Retirees' END AS age_band
        	, CASE WHEN
        		segment LIKE 'C%' THEN 'Couples'
         	ELSE 'Families' END AS demographic
         	, transactions
      	 	, ROUND(sales/transactions::numeric, 2) AS avg_transaction
         	, sales
	FROM data_mart.weekly_sales;
````

#### Answer

| week_date                  | week_number | month_number | year_number | region | platform | segment | age_band     | demographic | transactions | avg_transaction | sales    |
| ------------------------ | ----------- | ------------ | ----------- | ------ | -------- | ------- | ------------ | ----------- | ------------ | --------------- | -------- |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | ASIA   | Retail   | C3      | Retirees     | Couples     | 120631       | 30.31           | 3656163  |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | ASIA   | Retail   | F1      | Young Adults | Families    | 31574        | 31.56           | 996575   |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | USA    | Retail   | unknown | Retirees     | Families    | 529151       | 31.20           | 16509610 |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | EUROPE | Retail   | C1      | Young Adults | Couples     | 4517         | 31.42           | 141942   |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | AFRICA | Retail   | C2      | Middle Aged  | Couples     | 58046        | 30.29           | 1758388  |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | CANADA | Shopify  | F2      | Middle Aged  | Families    | 1336         | 182.54          | 243878   |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | AFRICA | Shopify  | F3      | Retirees     | Families    | 2514         | 206.64          | 519502   |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | ASIA   | Shopify  | F1      | Young Adults | Families    | 2158         | 172.11          | 371417   |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | AFRICA | Shopify  | F2      | Middle Aged  | Families    | 318          | 155.84          | 49557    |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020        | AFRICA | Retail   | C3      | Retirees     | Couples     | 111032       | 35.02           | 3888162  |

---

***

### 2. Data Exploration

**1. What day of the week is used for each week_date value?**

````sql
SELECT DISTINCT TO_CHAR(week_date, 'day') AS week_day
FROM clean_weekly_sales;
````

| week_day   |
| ---------- |
| monday     |

---

#### Answer
Each day used in the week_date value appears to be 'monday'.

***

**2. What range of week numbers are missing from the dataset?**

````sql

````

| dep_count  | avg_deposit |
| ---------- | ----------- |
| 5          | 508.61      |

---

#### Answer
blank

***

**3. How many total transactions were there for each year in the dataset?**

````sql

````

| txn_month | customer_count |
| --------- | -------------- |
| 1         | 53             |
| 2         | 36             |
| 3         | 38             |
| 4         | 22             |

---

#### Answer
blank

***

**4. What is the total sales for each region for each month?**

````sql

````

#### Code Explanation

***

**5. What is the total count of transactions for each platform?**

````sql

````

#### Answer
blank

***

**6. What is the percentage of sales for Retail vs Shopify for each month?**

````sql

````

#### Answer
blank

***

**7. What is the percentage of sales by demographic for each year in the dataset?**

````sql

````

#### Answer
blank

***

**8. Which age_band and demographic values contribute the most to Retail sales?**

````sql

````

#### Answer
blank

***

**9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**

````sql

````

#### Answer
blank

***

### 3. Before and After Analysis


** 

### 4. Bonus Question

***

