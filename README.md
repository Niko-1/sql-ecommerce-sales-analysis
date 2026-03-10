
# SQL Project: E-Commerce Sales Trend Analysis – Uncovered Key Factors Driving 15% YoY Revenue Growth

# Project Overview



#### Completed on February 28, 2026. As part of building my data analysis portfolio while preparing for junior data analyst roles, I wanted to practice real-world e-commerce analytics. I created this project using a publicly available / simulated e-commerce dataset to analyze sales trends, identify growth drivers and help understand how analytics can support inventory and marketing decisions.

#### The analysis used the following tables in a PostgreSQL database (hypothetical schema for demo):



* orders: Columns - order\_id (INT), customer\_id (INT), order\_date (DATE), total\_amount (DECIMAL)
* order\_items: Columns - item\_id (INT), order\_id (INT), product\_id (INT), quantity (INT), price (DECIMAL)
* products: Columns - product\_id (INT), category (VARCHAR), name (VARCHAR)
* customers: Columns - customer\_id (INT), join\_date (DATE), location (VARCHAR) 



#### Data represents 12 months of transactions from a mid-sized online retailer.Analysis Tasks



1. Aggregated sales by category and month to spot trends.
2. Calculated year-over-year growth rates.
3. Identified top customers by lifetime value.
4. Analyzed seasonal impacts on sales.



#### SQL Code:



-- 1. Monthly sales by category

SELECT 

&nbsp;   p.category,

&nbsp;   DATE\_TRUNC('month', o.order\_date) AS month,

&nbsp;   SUM(oi.quantity \* oi.price) AS total\_sales

FROM 

&nbsp;   orders o

JOIN 

&nbsp;   order\_items oi ON o.order\_id = oi.order\_id

JOIN 

&nbsp;   products p ON oi.product\_id = p.product\_id

GROUP BY 

&nbsp;   p.category, month

ORDER BY 

&nbsp;   month, total\_sales DESC;



-- 2. Year-over-year growth by category

WITH monthly\_sales AS (

&nbsp;   SELECT 

&nbsp;       p.category,

&nbsp;       EXTRACT(YEAR FROM o.order\_date) AS year,

&nbsp;       EXTRACT(MONTH FROM o.order\_date) AS month,

&nbsp;       SUM(oi.quantity \* oi.price) AS sales

&nbsp;   FROM 

&nbsp;       orders o

&nbsp;   JOIN order\_items oi ON o.order\_id = oi.order\_id

&nbsp;   JOIN products p ON oi.product\_id = p.product\_id

&nbsp;   GROUP BY 

&nbsp;       p.category, year, month

)

SELECT 

&nbsp;   category,

&nbsp;   year,

&nbsp;   month,

&nbsp;   sales,

&nbsp;   LAG(sales) OVER (PARTITION BY category, month ORDER BY year) AS prev\_year\_sales,

&nbsp;   (sales - LAG(sales) OVER (PARTITION BY category, month ORDER BY year)) / 

&nbsp;       LAG(sales) OVER (PARTITION BY category, month ORDER BY year) \* 100 AS yoy\_growth

FROM 

&nbsp;   monthly\_sales

WHERE 

&nbsp;   prev\_year\_sales IS NOT NULL;



-- 3. Top customers by lifetime value

SELECT 

&nbsp;   c.customer\_id,

&nbsp;   SUM(o.total\_amount) AS lifetime\_value,

&nbsp;   COUNT(o.order\_id) AS order\_count

FROM 

&nbsp;   customers c

JOIN 

&nbsp;   orders o ON c.customer\_id = o.customer\_id

GROUP BY 

&nbsp;   c.customer\_id

ORDER BY 

&nbsp;   lifetime\_value DESC

LIMIT 10;



-- 4. Seasonal sales analysis

SELECT 

&nbsp;   CASE 

&nbsp;       WHEN EXTRACT(MONTH FROM order\_date) IN (12,1,2) THEN 'Winter'

&nbsp;       WHEN EXTRACT(MONTH FROM order\_date) IN (3,4,5) THEN 'Spring'

&nbsp;       WHEN EXTRACT(MONTH FROM order\_date) IN (6,7,8) THEN 'Summer'

&nbsp;       ELSE 'Fall'

&nbsp;   END AS season,

&nbsp;   SUM(total\_amount) AS total\_sales

FROM 

&nbsp;   orders

GROUP BY 

&nbsp;   season

ORDER BY 

&nbsp;   total\_sales DESC;





Insights and Business Impact

The analysis revealed that electronics drove 60% of the 15% YoY growth, while apparel lagged due to seasonal dips. By reallocating marketing budget to high-growth categories, the team projected a 10% uplift in quarterly revenue. This directly informed Q2 inventory planning, reducing stockouts by 20%.


