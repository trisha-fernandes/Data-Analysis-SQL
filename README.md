# Data-Analysis-SQL
## 1. Total Orders
```sql
SELECT
  COUNT(*) AS orders

FROM
  `bigquery-public-data.thelook_ecommerce.orders`
```
<img width="369" height="110" alt="image" src="https://github.com/user-attachments/assets/ada83cc4-a6f9-47c0-bdab-3d8f8b72a00e" />

### Insights: The total number of orders are 124,900.


## 2. Total Products
```sql
SELECT
  COUNT(*) AS products

FROM
  `bigquery-public-data.thelook_ecommerce.products`
```
<img width="379" height="110" alt="image" src="https://github.com/user-attachments/assets/8fdf30d2-d8bf-424b-92b5-12a9227cab55" />

### Insights: 

## 3. Total Customers
```sql
SELECT
  COUNT(*) AS customers

FROM
  `bigquery-public-data.thelook_ecommerce.users`
```
<img width="379" height="111" alt="image" src="https://github.com/user-attachments/assets/6117d5a3-bd2e-4294-8a0e-7feb1a88b88c" />

### Insights: 

## 4. Monthly Gross Revenue Trend (2024, 2025 & 2026)
```sql
SELECT
  FORMAT_DATETIME('%Y-%m', created_at) AS month,
  ROUND(SUM(sale_price),2) AS revenue

FROM
  `bigquery-public-data.thelook_ecommerce.order_items`

WHERE
  EXTRACT(YEAR FROM created_at) IN (2024, 2025, 2026)

GROUP BY
  month
```
<img width="900" height="665" alt="image" src="https://github.com/user-attachments/assets/0e871c25-0a7b-42ae-8076-fae2b0062eb8" />

### Insights: 

## 5. Top 10 Products by Quantity Sold
```sql
SELECT
  p.name,
  COUNT(*) AS quantity_sold,
  RANK() OVER(
    ORDER BY COUNT(*) DESC
  ) AS product_rank_by_quantity,
  ROUND(AVG(p.retail_price),2) AS retail_price
  
FROM `bigquery-public-data.thelook_ecommerce.order_items` oi  
JOIN `bigquery-public-data.thelook_ecommerce.products` p   
ON oi.product_id = p.id

GROUP BY p.name
ORDER BY quantity_sold DESC
LIMIT 10;
```
<img width="1314" height="488" alt="image" src="https://github.com/user-attachments/assets/94e893d1-60cd-438e-b4e4-2f98cc32e880" />

### Insights: 

## 6. Top 10 Products by Revenue
```sql
SELECT
  p.name,
  ROUND(SUM(oi.sale_price),2) AS revenue,
  ROUND(AVG(oi.sale_price),2) AS retail_price,
  COUNT(*) AS quantity_sold
  
FROM
  `bigquery-public-data.thelook_ecommerce.order_items` oi
  
JOIN
  `bigquery-public-data.thelook_ecommerce.products` p 
  
ON 
  oi.product_id = p.id

GROUP BY
  p.name

ORDER BY
  revenue DESC

LIMIT
  10;
```
<img width="1251" height="524" alt="image" src="https://github.com/user-attachments/assets/a5b32192-5752-475e-bb97-e6fd9c7d4f11" />


### Insights: 

## 7. Top 10 cusomters by Revenue
```sql
SELECT
  u.id,
  CONCAT(u.first_name,' ',u.last_name) AS full_name,
  ROUND(SUM(oi.sale_price),2) AS revenue,
  ROUND(AVG(oi.sale_price),2) AS average_order_value,
  COUNT(*) AS total_orders

FROM  `bigquery-public-data.thelook_ecommerce.users` u
JOIN  `bigquery-public-data.thelook_ecommerce.order_items` oi
ON  u.id = oi.user_id

GROUP BY  u.id,full_name
ORDER BY revenue DESC
LIMIT  10;
```
<img width="1279" height="482" alt="image" src="https://github.com/user-attachments/assets/9f5c2e77-9c07-4acf-aaa0-2064d30ae739" />

### Insights: 

## 8. Monthly Running Revenue Partitioned by Year
```sql
WITH monthly_revenue AS (
    SELECT
        DATE_TRUNC(DATE(created_at), MONTH) AS revenue_month,
        ROUND(SUM(sale_price),2) AS monthly_total
    FROM `bigquery-public-data.thelook_ecommerce.order_items`
    WHERE
        EXTRACT(YEAR FROM created_at) IN (2025, 2026)
    GROUP BY revenue_month
)

SELECT
    revenue_month,
    monthly_total,
    ROUND(SUM(monthly_total) OVER (
        PARTITION BY EXTRACT(YEAR FROM revenue_month)
        ORDER BY revenue_month
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ),2) AS yearly_running_revenue,
ROUND(LAG(monthly_total) OVER (ORDER BY revenue_month),2) AS previous_month_revenue,
ROUND(
    100 * (
        monthly_total -
        LAG(monthly_total) OVER (ORDER BY revenue_month)
    ) /
    LAG(monthly_total) OVER (ORDER BY revenue_month),
    2
) AS mom_growth_pct

FROM monthly_revenue
ORDER BY revenue_month;
```
<img width="962" height="640" alt="image" src="https://github.com/user-attachments/assets/4f3dcb6a-dcb9-49b7-aa03-119e024ed212" />


### Insights: 

## 9. Customer Segmentation
```sql
WITH customer_spending AS (
  SELECT
    user_id,
    SUM(sale_price) AS revenue

  FROM `bigquery-public-data.thelook_ecommerce.order_items`
  GROUP BY
    user_id
)

SELECT
  CASE
    WHEN customer_spending.revenue > 500 THEN "High Value"
    WHEN customer_spending.revenue > 200 THEN "Medium Value"
    ELSE "Low Value"
  END AS segment,
  COUNT(*) AS total_customers

FROM  customer_spending
GROUP BY
  segment
```
<img width="876" height="241" alt="image" src="https://github.com/user-attachments/assets/124269dc-0560-424f-9f77-32d39bcaf43b" />

### Insights: 

## 10. Repeat Customers
```sql
WITH cusomter_orders AS (
  SELECT
    user_id,
    COUNT(order_id) AS orders
  FROM
    `bigquery-public-data.thelook_ecommerce.orders`
  
  GROUP BY
    user_id
)

SELECT
  COUNTIF(cusomter_orders.orders > 1) AS repeat_customers,
```
<img width="485" height="82" alt="image" src="https://github.com/user-attachments/assets/0ffa3305-9c71-4cd0-a04d-ef11582af973" />

### Insights: 

## 11. Top Product in each Category
```sql
SELECT
p.category AS category,
p.name AS name,
ROUND(SUM(sale_price),2) AS revenue

FROM
`bigquery-public-data.thelook_ecommerce.order_items` oi
JOIN
`bigquery-public-data.thelook_ecommerce.products` p
ON
oi.product_id = p.id

GROUP BY
  category,
  name

QUALIFY(
RANK() OVER(
  PARTITION BY category
  ORDER BY revenue DESC
))=1
```
<img width="660" height="371" alt="image" src="https://github.com/user-attachments/assets/8a8f5176-7ed0-4640-8e76-ec99537e6bf3" />
<img width="658" height="813" alt="image" src="https://github.com/user-attachments/assets/083ae68a-1438-4ad5-a035-10b87a719d7a" />

### Insights: 

