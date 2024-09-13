# Coffee Shop Sales Analysis

## Table of Content

- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Result and Findings](#result-and-findings)
- [Recommendations](#recommendations)
- [Limitations](#limitations)


### Project Overview
---

A data analysis project utilizing MySQL and Power BI revealed the company's coffee sales for 6 months (January to June 2023). Creating a comprehensive sales analysis dashboard to provide actionable insights into sales performance, trends, and opportunities for optimization.

![dashboard](https://github.com/user-attachments/assets/c9b7124e-cc01-4f4c-86f1-0bee8d30f879)


### Data Source
---

Coffee Shop Sale: The primary dataset used for this analysis is the "Coffee Shop Sales.csv" file, containing sales data, order information, product details, and store locations.

### Tools
---

- MySQL Server | data extraction and management.
- Power BI | data modeling, visualization, and dashboard creation.

### Data Cleaning and Preparation
---

In the initial data preparation phase, I performed the following tasks:
1. Data Extraction: Extracting relevant data from the MySQL Server database, including sales data, order information, product details, and store locations.
2. Data Cleaning: Addressing data quality issues such as missing values, duplicates, and inconsistencies.
3. Data Transformation: Transforming data into a suitable format for analysis, including creating derived columns and aggregating data.

### Exploratory Data Analysis
---

EDA involves exploring the data to answer key questions such as:
1. Data Profiling: Understanding the characteristics of the data, including data types, distributions, and summary statistics.
2. Data Visualization: Creating visualizations to explore relationships between variables and identify potential patterns.
3. Outlier Detection: Identifying and handling outliers that may skew the analysis.

### Data Analysis
---
1. KPI Calculation: Calculated key performance indicators (KPIs) as outlined in the problem statement, including total sales, total orders, total quantity sold, month-on-month growth, and differences.
2. Trend Analysis: Analyzed trends in sales data over time, identifying seasonal patterns, growth rates, and fluctuations.
3. Product Analysis: Evaluated sales performance across different product categories, identifying top-selling items and areas for improvement.
4. Store Analysis: Compared sales performance across different store locations, identifying high-performing stores and areas with growth potential.

```SQL Queries
select * from coffee_shop_sales;
describe coffee_shop_sales;
set SQL_SAFE_UPDATES = 0;

UPDATE coffee_shop_sales
SET transaction_time = STR_TO_DATE(transaction_time, '%H:%i:%s');

ALTER TABLE coffee_shop_sales
CHANGE COLUMN ï»¿transaction_id transaction_id INT;

-- TOTAL SALES
SELECT CONCAT((ROUND(SUM(unit_price * transaction_qty)))/ 1000, "K") as Total_Sales 
FROM coffee_shop_sales 
WHERE MONTH(transaction_date) = 5 -- for month of (CM-May)
;

-- TOTAL SALES KPI - MOM DIFFERENCE AND MOM GROWTH
SELECT 
    MONTH(transaction_date) AS month,
    ROUND(SUM(unit_price * transaction_qty)) AS total_sales,
    (SUM(unit_price * transaction_qty) - LAG(SUM(unit_price * transaction_qty), 1)
    OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(unit_price * transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) IN (4, 5) -- for months of April and May
GROUP BY 
    MONTH(transaction_date)
ORDER BY 
    MONTH(transaction_date);

-- TOTAL ORDERS
SELECT COUNT(transaction_id) as Total_Orders
FROM coffee_shop_sales 
WHERE MONTH (transaction_date)= 5 -- for month of (CM-May)
;
    
    
-- TOTAL ORDERS KPI - MOM DIFFERENCE AND MOM GROWTH
SELECT 
    MONTH(transaction_date) AS month,
    ROUND(COUNT(transaction_id)) AS total_orders,
    (COUNT(transaction_id) - LAG(COUNT(transaction_id), 1) 
    OVER (ORDER BY MONTH(transaction_date))) / LAG(COUNT(transaction_id), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) IN (4, 5) -- for April and May
GROUP BY 
    MONTH(transaction_date)
ORDER BY 
    MONTH(transaction_date);
    
    
-- TOTAL QUANTITY SOLD
SELECT SUM(transaction_qty) as Total_Quantity_Sold
FROM coffee_shop_sales 
WHERE MONTH(transaction_date) = 5 -- for month of (CM-May)
;

-- TOTAL QUANTITY SOLD KPI - MOM DIFFERENCE AND MOM GROWTH
SELECT 
    MONTH(transaction_date) AS month,
    ROUND(SUM(transaction_qty)) AS total_quantity_sold,
    (SUM(transaction_qty) - LAG(SUM(transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) IN (4, 5)   -- for April and May
GROUP BY 
    MONTH(transaction_date)
ORDER BY 
    MONTH(transaction_date);
    

-- CALENDAR TABLE – DAILY SALES, QUANTITY and TOTAL ORDERS
SELECT
    SUM(unit_price * transaction_qty) AS total_sales,
    SUM(transaction_qty) AS total_quantity_sold,
    COUNT(transaction_id) AS total_orders
FROM 
    coffee_shop_sales
WHERE 
    transaction_date = '2023-05-18'; -- For 18 May 2023

-- If you want to get exact Rounded off values then use below query to get the result:
SELECT
    Concat(ROUND(SUM(unit_price * transaction_qty)/1000, 1),'K') AS total_sales,
    Concat(ROUND(SUM(transaction_qty)/1000, 1), 'k') AS total_quantity_sold,
    Concat(ROUND(COUNT(transaction_id)/1000, 1), 'k') AS total_orders
FROM 
    coffee_shop_sales
WHERE 
    transaction_date = '2023-05-18'; -- For 18 May 2023
    

-- SALES BY WEEKDAY / WEEKEND:
SELECT 
    CASE 
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekends'
        ELSE 'Weekdays'
    END AS day_type,
    CONCAT(ROUND(SUM(unit_price * transaction_qty)/ 1000,1), 'K') AS total_sales
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) = 5  -- Filter for May
GROUP BY 
    CASE 
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekends'
        ELSE 'Weekdays'
    END;
    
-- SALES BY STORE LOCATION
SELECT 
	store_location,
	CONCAT(ROUND(SUM(unit_price * transaction_qty)/1000, 2), 'K') as Total_Sales
FROM coffee_shop_sales
WHERE
	MONTH(transaction_date) =5 
GROUP BY store_location
ORDER BY 	SUM(unit_price * transaction_qty) DESC;


-- SALES TREND OVER PERIOD
SELECT
CONCAT(ROUND(AVG(total_sales)/1000, 1), 'K') AS average_sales
FROM (
    SELECT 
        SUM(unit_price * transaction_qty) AS total_sales
    FROM 
        coffee_shop_sales
	WHERE 
        MONTH(transaction_date) = 5  -- Filter for May
    GROUP BY 
        transaction_date
) AS internal_query;

-- DAILY SALES FOR MONTH SELECTED
SELECT 
    DAY(transaction_date) AS day_of_month,
    ROUND(SUM(unit_price * transaction_qty),1) AS total_sales
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) = 5  -- Filter for May
GROUP BY 
    DAY(transaction_date)
ORDER BY 
    DAY(transaction_date);
    
    
-- COMPARING DAILY SALES WITH AVERAGE SALES – IF GREATER THAN “ABOVE AVERAGE” and LESSER THAN “BELOW AVERAGE”
SELECT 
    day_of_month,
    CASE 
        WHEN total_sales > avg_sales THEN 'Above Average'
        WHEN total_sales < avg_sales THEN 'Below Average'
        ELSE 'Average'
    END AS sales_status,
    total_sales
FROM (
    SELECT 
        DAY(transaction_date) AS day_of_month,
        SUM(unit_price * transaction_qty) AS total_sales,
        AVG(SUM(unit_price * transaction_qty)) OVER () AS avg_sales
    FROM 
        coffee_shop_sales
    WHERE 
        MONTH(transaction_date) = 5  -- Filter for May
    GROUP BY 
        DAY(transaction_date)
) AS sales_data
ORDER BY 
    day_of_month;


-- SALES BY PRODUCT CATEGORY
SELECT 
	product_category,
	ROUND(SUM(unit_price * transaction_qty),1) as Total_Sales
FROM coffee_shop_sales
WHERE
	MONTH(transaction_date) = 5 
GROUP BY product_category
ORDER BY SUM(unit_price * transaction_qty) DESC;


-- SALES BY PRODUCTS (TOP 10)
SELECT 
	product_type,
	ROUND(SUM(unit_price * transaction_qty),1) as Total_Sales
FROM coffee_shop_sales
WHERE
	MONTH(transaction_date) = 5 AND product_category = 'Coffee'
GROUP BY product_type
ORDER BY SUM(unit_price * transaction_qty) DESC
LIMIT 10;


-- SALES BY DAY | HOUR
SELECT 
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales,
    SUM(transaction_qty) AS Total_Quantity,
    COUNT(*) AS Total_Orders
FROM 
    coffee_shop_sales
WHERE 
    DAYOFWEEK(transaction_date) = 3 -- Filter for Tuesday (1 is Sunday, 2 is Monday, ..., 7 is Saturday)
    AND HOUR(transaction_time) = 8 -- Filter for hour number 8
    AND MONTH(transaction_date) = 5; -- Filter for May (month number 5)


-- TO GET SALES FROM MONDAY TO SUNDAY FOR MONTH OF MAY
SELECT 
    CASE 
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
    END AS Day_of_Week,
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) = 5 -- Filter for May (month number 5)
GROUP BY 
    CASE 
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
    END;

-- TO GET SALES FOR ALL HOURS FOR MONTH OF MAY
SELECT 
    HOUR(transaction_time) AS Hour_of_Day,
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales
FROM 
    coffee_shop_sales
WHERE 
    MONTH(transaction_date) = 5 -- Filter for May (month number 5)
GROUP BY 
    HOUR(transaction_time)
ORDER BY 
    HOUR(transaction_time);

```

### Result and Findings
---

The analysis results are as follows:
1.	Total sales increased by 35.3% in the second quarter compared to the first quarter.
2.	The coffee product category has consistently outperformed other categories
3.	The Hell's Kitchen store location has experienced a consistent increase sales in recent months
4.	The top reasons for cart abandonment are no guest checkout option, complex checkout, no total order upfront, and shipping costs.
5.	Most abandoned users come from social media (44-42%) and the session range where abandonment is highest is between 77-100 sessions
6.	The cart contents with the highest abandonment are accessories, toys, and footwear
7.	The majority of abandoned users are female (54.35%)
8.	The location date suggests that abandonment is spread across several regions in North America with Virginia being notably higher.
9.	The trend of abandoned users by week shows fluctuation with some peaks suggesting periodic increases in abandonment rates
10.	items such as sunglasses, wristwatches, and tablets are among the top categories for abandoned purchases

### Recommendations
---

1.	Continue current marketing strategies and explore opportunities for further growth.
2.	Allocate more resources to promote and expand the coffee product category.
3.	Conduct a market analysis to identify reasons for the decline and implement corrective measures in other store locations.
4.	Leverage Social Media: Since a significant portion of traffic comes from social media, retargeting campaigns on these platforms could help recover abandoned carts.
5.	Tailored Marketing Strategies: Develop targeted marketing strategies for the top abandoned items like sunglasses, wristwatches, and tablets to improve conversions in these categories
6.	Address Shipping Costs: Offering free shipping thresholds or flat-rate shipping could mitigate abandonment due to shipping costs.

### Limitations
---

1. Data Quality: The quality of the data impacts the accuracy and reliability of the analysis.
2. Data Availability: Limited data availability restricts the scope of the analysis.
3. Assumptions: Certain assumptions made during the analysis, could affect the results.
4. Complexity: The complexity of the data and the analysis introduce challenges and potential biases.

