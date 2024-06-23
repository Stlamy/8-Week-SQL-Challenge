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
SELECT DISTINCT EXTRACT(WEEK FROM week_date) AS week_numbers
FROM clean_weekly_sales
ORDER BY week_numbers;
````

| week_numbers |
| ------------ |
| 13           |
| 14           |
| 15           |
| 16           |
| 17           |
| 18           |
| 19           |
| 20           |
| 21           |
| 22           |
| 23           |
| 24           |
| 25           |
| 26           |
| 27           |
| 28           |
| 29           |
| 30           |
| 31           |
| 32           |
| 33           |
| 34           |
| 35           |
| 36           |

---

#### Answer
Looking at the query above, notice that there are two gaps in the week numbers available in the data. There are 52 weeks in a year, so currently, Data Mart is missing data for weeks 1-12 and 37-52, or 28 weeks in total.

***

**3. How many total transactions were there for each year in the dataset?**

````sql
SELECT
	year_number
    	, SUM(transactions) AS transaction_count
FROM clean_weekly_sales
GROUP BY year_number
ORDER BY year_number;
````

| year_number | transaction_count |
| ----------- | ----------------- |
| 2018        | 346406460         |
| 2019        | 365639285         |
| 2020        | 375813651         |

---

#### Answer
From the table resulting from the query above, notice the following count of transactions by calendar year:
- 2018 = 346,406,460
- 2019 = 365,639,285
- 2020 = 375,813,651

***

**4. What is the total sales for each region for each month?**

````sql
SELECT
	region
	, month_number
    	, SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY region, month_number
ORDER BY region, month_number;
````

| region        | month_number | total_sales |
| ------------- | ------------ | ----------- |
| AFRICA        | 3            | 567767480   |
| AFRICA        | 4            | 1911783504  |
| AFRICA        | 5            | 1647244738  |
| AFRICA        | 6            | 1767559760  |
| AFRICA        | 7            | 1960219710  |
| AFRICA        | 8            | 1809596890  |
| AFRICA        | 9            | 276320987   |
| ASIA          | 3            | 529770793   |
| ASIA          | 4            | 1804628707  |
| ASIA          | 5            | 1526285399  |
| ASIA          | 6            | 1619482889  |
| ASIA          | 7            | 1768844756  |
| ASIA          | 8            | 1663320609  |
| ASIA          | 9            | 252836807   |
| CANADA        | 3            | 144634329   |
| CANADA        | 4            | 484552594   |
| CANADA        | 5            | 412378365   |
| CANADA        | 6            | 443846698   |
| CANADA        | 7            | 477134947   |
| CANADA        | 8            | 447073019   |
| CANADA        | 9            | 69067959    |
| EUROPE        | 3            | 35337093    |
| EUROPE        | 4            | 127334255   |
| EUROPE        | 5            | 109338389   |
| EUROPE        | 6            | 122813826   |
| EUROPE        | 7            | 136757466   |
| EUROPE        | 8            | 122102995   |
| EUROPE        | 9            | 18877433    |
| OCEANIA       | 3            | 783282888   |
| OCEANIA       | 4            | 2599767620  |
| OCEANIA       | 5            | 2215657304  |
| OCEANIA       | 6            | 2371884744  |
| OCEANIA       | 7            | 2563459400  |
| OCEANIA       | 8            | 2432313652  |
| OCEANIA       | 9            | 372465518   |
| SOUTH AMERICA | 3            | 71023109    |
| SOUTH AMERICA | 4            | 238451531   |
| SOUTH AMERICA | 5            | 201391809   |
| SOUTH AMERICA | 6            | 218247455   |
| SOUTH AMERICA | 7            | 235582776   |
| SOUTH AMERICA | 8            | 221166052   |
| SOUTH AMERICA | 9            | 34175583    |
| USA           | 3            | 225353043   |
| USA           | 4            | 759786323   |
| USA           | 5            | 655967121   |
| USA           | 6            | 703878990   |
| USA           | 7            | 760331754   |
| USA           | 8            | 712002790   |
| USA           | 9            | 110532368   |

---

***

Observe the resulting table from the query above for total sales by region and month. All continents/countries provided in the data set have sales recorded between March and September.

**5. What is the total count of transactions for each platform?**

````sql
SELECT
	platform
	, SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY platform
ORDER BY platform;
````

| platform    | total_transactions |
| ----------- | -----------------  |
| Retail      | 1081934227         |
| Shopify     | 5925169            |

---

#### Answer
Retail sales outpace Shopify sales by over 180 percentage points and represent the majority share of transactions for Data Mart.

***

**6. What is the percentage of sales for Retail vs Shopify for each month?**

````sql
WITH cte_retail AS (
  SELECT
	year_number
	, month_number
  	, SUM(sales) AS retail_sales
  FROM clean_weekly_sales
  WHERE platform LIKE 'Retail'
  GROUP BY year_number, month_number
  ORDER BY year_number, month_number)
, cte_shopify AS (
  SELECT
  	year_number
	, month_number
  	, SUM(sales) AS shopify_sales
  FROM clean_weekly_sales
  WHERE platform LIKE 'Shopify'
  GROUP BY year_number, month_number
  ORDER BY year_number, month_number)
, cte_total AS (
  SELECT
	year_number
	, month_number
  	, SUM(sales) AS total_sales
  FROM clean_weekly_sales
  GROUP BY year_number, month_number
  ORDER BY year_number, month_number)

SELECT
	a.year_number
	, a.month_number
	, ROUND((b.retail_sales * 100.0) / a.total_sales, 2) AS retail_share
    	, ROUND((c.shopify_sales  * 100.0) / a.total_sales, 2) AS shopify_share
FROM cte_total a
LEFT JOIN cte_retail b
ON a.year_number = b.year_number AND a.month_number = b.month_number
LEFT JOIN cte_shopify c
ON a.year_number = c.year_number AND a.month_number = c.month_number
ORDER BY a.year_number, a.month_number;
````


| year_number | month_number | retail_share | shopify_share |
| ----------- | ------------ | ------------ | ------------- |
| 2018        | 3            | 97.92        | 2.08          |
| 2018        | 4            | 97.93        | 2.07          |
| 2018        | 5            | 97.73        | 2.27          |
| 2018        | 6            | 97.76        | 2.24          |
| 2018        | 7            | 97.75        | 2.25          |
| 2018        | 8            | 97.71        | 2.29          |
| 2018        | 9            | 97.68        | 2.32          |
| 2019        | 3            | 97.71        | 2.29          |
| 2019        | 4            | 97.80        | 2.20          |
| 2019        | 5            | 97.52        | 2.48          |
| 2019        | 6            | 97.42        | 2.58          |
| 2019        | 7            | 97.35        | 2.65          |
| 2019        | 8            | 97.21        | 2.79          |
| 2019        | 9            | 97.09        | 2.91          |
| 2020        | 3            | 97.30        | 2.70          |
| 2020        | 4            | 96.96        | 3.04          |
| 2020        | 5            | 96.71        | 3.29          |
| 2020        | 6            | 96.80        | 3.20          |
| 2020        | 7            | 96.67        | 3.33          |
| 2020        | 8            | 96.51        | 3.49          |

---

#### Answer
From the table resulting from the query above, Shopify sales represent at most 3.49% of total share by month for Data Mart across all months available in the dataset. However, notice that the Shopify share of sales increasing over time.

***

**7. What is the percentage of sales by demographic for each year in the dataset?**

````sql
WITH cte_couple AS (
  SELECT
	year_number
  	, SUM(sales) AS couple_sales
  FROM clean_weekly_sales
  WHERE demographic LIKE 'Couples'
  GROUP BY year_number
  ORDER BY year_number)
, cte_families AS (
  SELECT
  	year_number
  	, SUM(sales) AS family_sales
  FROM clean_weekly_sales
  WHERE demographic LIKE 'Families'
  GROUP BY year_number
  ORDER BY year_number)
, cte_total AS (
  SELECT
	year_number
  	, SUM(sales) AS total_sales
  FROM clean_weekly_sales
  GROUP BY year_number
  ORDER BY year_number)

SELECT
	a.year_number
	, ROUND((b.couple_sales * 100.0) / a.total_sales, 2) AS couple_share
    	, ROUND((c.family_sales  * 100.0) / a.total_sales, 2) AS family_share
FROM cte_total a
LEFT JOIN cte_couple b
ON a.year_number = b.year_number
LEFT JOIN cte_families c
ON a.year_number = c.year_number
ORDER BY a.year_number;
````

| year_number | couple_share | family_share |
| ----------- | ------------ | ------------ |
| 2018        | 26.38        | 73.62        |
| 2019        | 27.28        | 72.72        |
| 2020        | 28.72        | 71.28        |

---

#### Answer
From the resulting table from the SQL query above, families represent the largest share of sales by year for Data Mart. Couple share has been increasing over time, but they still represent under 30% of total yearly sales. This seems intuitive, as family households are larger than those for couples and should result in greater spending.

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

