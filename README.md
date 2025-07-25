# Retail Sales Analysis – SQL Project

This project performs **end-to-end data analysis** on a retail sales dataset using **MySQL**. It demonstrates **data cleaning, exploration, and solving key business problems** using SQL queries. The goal is to derive actionable insights such as **top customers, best-selling categories, seasonal trends, and profit margins**.

---

## 📂 Project Overview

- **Database Used:** MySQL  
- **Dataset:** Retail sales transactions (customer demographics, sales amount, category, quantity, etc.)  
- **Objective:** Analyze sales data to answer business questions like:
  - Which categories generate maximum revenue?
  - Who are the top customers?
  - What are the best selling months and peak sales hours?
  - What is the profit margin per category?

---

## Steps Performed

# 1. Database Creation & Table Setup
```sql
CREATE DATABASE sql_project_p2;
USE sql_project_p2;

CREATE TABLE retail_sales (
   transactions_id INT PRIMARY KEY,
   sale_date DATE,
   sale_time TIME,
   customer_id INT,
   gender VARCHAR(15),
   age INT,
   category VARCHAR(15),
   quantity INT,
   price_per_unit FLOAT,
   cogs FLOAT,
   total_sale FLOAT
);


#data cleaning

select * from 
retail_sales
where 
	transactions_id is null
      or
	sale_date is null
      or
	sale_time is null
  or
  customer_id is null
  or
   gender is null
   or
   age is null
   or
   category	is null
   or
   quantity	 is null
   or
   price_per_unit is null
     or
   cogs is null
   or
   total_sale is null;

#data exploration
select count(*) as total_sales from retail_sales;

select count(distinct customer_id) as customers from retail_sales;
select * from  retail_sales;

#data analysis/business problem
-- Data Analysis & Business Key Problems & Answers

-- My Analysis & Findings
-- Q.1 Write a SQL query to retrieve all columns for sales made on '2022-11-05
 select count(distinct customer_id)
from retail_sales
where sale_date = "2022-11-05";







-- Q.2 Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 10 in the month of Nov-2022




SELECT *
FROM retail_sales
WHERE category = 'Clothing'
  AND DATE_FORMAT(sale_date, '%Y-%m') = '2022-11'
  AND quantity>= 4;

#another method
SELECT *
FROM retail_sales
WHERE category = 'Clothing'
  AND extract(MONTH from sale_date ) = '11'
  AND EXTRACT(YEAR FROM sale_date) = '2022'
  AND quantity>= 4;










--# Q.3 Write a SQL query to calculate the total sales (total_sale) for each category.

 
 
 SELECT 
    category,
    SUM(total_sale) as net_sale
FROM retail_sales
GROUP BY 1;

select category, sum(total_sale),
count(*) as total_orders
 from retail_sales
 group by category;





-- Q.4 Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.



select  avg(age)
from retail_sales
where category = 'Beauty';







-- Q.5 Write a SQL query to find all transactions where the total_sale is greater than 1000.




select * 
   from retail_sales
   where total_sale>1000;

-- Q.6 Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.

select gender, category, count(*) as total_transaction
from retail_sales
group by gender, category;
   





-- Q.7 Write a SQL query to calculate the average sale for each month. Find out best selling month in each year

SELECT 
    year,
    month,
    avg_sale
FROM 
(
    SELECT 
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        AVG(total_sale) AS avg_sale,
        RANK() OVER(
            PARTITION BY EXTRACT(YEAR FROM sale_date) 
            ORDER BY AVG(total_sale) DESC
        ) AS rank_no
    FROM retail_sales
    GROUP BY year, month
) AS t1
WHERE rank_no = 1;











-- Q.8 Write a SQL query to find the top 5 customers based on the highest total sales 
 select distinct customer_id , sum(total_sale) as total_sale
 from retail_sales
 group by 1
 order by 2 desc
 limit 5;






-- Q.9 Write a SQL query to find the number of unique customers who purchased items from each category.

 select distinct count(distinct customer_id) as customers, category
 from retail_sales
 group by 2;
 
 







-- Q.10 Write a SQL query to create each shift and number of orders (Example Morning <=12, Afternoon Between 12 & 17, Evening >17)


WITH hourly_sale
AS
(
SELECT *,
    CASE
        WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END as shift
FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) as total_orders    
FROM hourly_sale
GROUP BY shift;

#another method

SELECT count(*) as total_orders,
    CASE
        WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END as shift
FROM retail_sales
group by  shift;




#Customer Lifetime Value (CLV):

select customer_id, sum(total_sale) as lifetime_value
from retail_sales
group by customer_id
order by lifetime_value desc;

#Overall AOV (not per customer):
SELECT SUM(total_sale)/COUNT(distinct(transactions_id)) AS avg_order_value
FROM retail_sales;

#AOV per customer:
SELECT 
    customer_id,
    SUM(total_sale) / COUNT(DISTINCT transactions_id) AS aov
FROM retail_sales
GROUP BY customer_id
ORDER BY aov DESC;

#Repeat Customer Rate:
SELECT 
    COUNT(DISTINCT customer_id) - COUNT(DISTINCT CASE WHEN order_count=1 THEN customer_id END) AS repeat_customers
FROM (
    SELECT customer_id, COUNT(*) AS order_count
    FROM retail_sales
    GROUP BY customer_id
) t;

#Gross Profit per Category (Total Sales - COGS):

SELECT category, SUM(total_sale - cogs) AS gross_profit
FROM retail_sales
GROUP BY category;

#Profit Margin % by Category:
SELECT category,
       SUM(total_sale - cogs)/SUM(total_sale)*100 AS profit_margin_pct
FROM retail_sales
GROUP BY category;

#Age Group Analysis:
SELECT 
  CASE 
    WHEN age < 20 THEN 'Teen'
    WHEN age BETWEEN 20 AND 35 THEN 'Young Adult'
    WHEN age BETWEEN 36 AND 50 THEN 'Adult'
    ELSE 'Senior'
  END AS age_group,
  SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY age_group;


#Festival/Peak Season Sales (e.g., Oct-Dec)

SELECT category, SUM(total_sale) AS festive_sales, sale_date
FROM retail_sales
WHERE MONTH(sale_date) IN (10,11,12)
GROUP BY category,sale_date;




#Peak Sales Hours:
SELECT EXTRACT(HOUR FROM sale_time) AS hour, COUNT(*) AS orders
FROM retail_sales
GROUP BY hour
ORDER BY orders DESC;

#Average of all hours

SELECT avg(EXTRACT(HOUR FROM sale_time)) AS avghour, COUNT(*) AS orders
FROM retail_sales
ORDER BY orders DESC;

#Order Distribution by Day of Week

SELECT DAYNAME(sale_date) AS day_of_week, COUNT(*) AS orders
FROM retail_sales
GROUP BY day_of_week
ORDER BY orders DESC;



#Author - Gourav Das

linkedin -' https://www.linkedin.com/in/gourav-das-103a6424a/'
