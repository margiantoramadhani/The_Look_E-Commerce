# The_Look_E-Commerce
Data analytics the look e-commerce dataset by SQL
Dataset : https://console.cloud.google.com/bigquery?project=swift-handler-377010&ws=!1m4!1m3!3m2!1ssql-project-376612!2sthelook_ecommerce
----

To understand which categories is the best sellers amongs all, Please find the categories that generate the most sales price with status shipped on Dec 2022?

```sql
SELECT 
  SUM(sale_price) AS total_price, product_category
FROM
  (
  SELECT 
    o.product_id, o.status, DATE(o.shipped_at), o. sale_price, i.product_category
  FROM 
    sql-project-376612.thelook_ecommerce.order_items AS o
  JOIN 
    sql-project-376612.thelook_ecommerce.inventory_items AS i
  ON 
    o.product_id = i.product_id 
  WHERE 
    o.status = 'Shipped' AND DATE(o.shipped_at) BETWEEN '2022-12-01' AND '2022-12-31'
  )
GROUP BY product_category ORDER BY SUM(sale_price) DESC
```

----

Which category faced lowest % (most shrinking) in terms of order number with status shipped on Dec 2022 compared to previous month? 


```sql
WITH order_number_dec as
  (
  SELECT 
    p.category,
    COUNT(DISTINCT o.id) AS total_order_dec
  FROM 
    sql-project-376612.thelook_ecommerce.order_items AS o
  JOIN 
    sql-project-376612.thelook_ecommerce.products AS p
  ON  
    p.id = o.product_id
  WHERE EXTRACT(month FROM shipped_at) = 12 and status ='Shipped'
  GROUP BY 1
  ORDER BY 2 DESC
  ),

order_number_nov as 
  (
  SELECT 
    p.category,
    COUNT(DISTINCT o.id) AS total_order_nov
  FROM 
    sql-project-376612.thelook_ecommerce.order_items AS o
  JOIN 
    sql-project-376612.thelook_ecommerce.products AS p
  ON  
    p.id = o.product_id
  WHERE EXTRACT(month FROM shipped_at) = 11 AND status ='Shipped'
  GROUP BY 1
  ORDER BY 2 DESC
  ),

main as
  (
  SELECT
    order_number_dec.category,
    order_number_dec.total_order_dec,
    order_number_nov.total_order_nov,
    ((order_number_dec.total_order_dec - order_number_nov.total_order_nov)/order_number_nov.total_order_nov)*100 AS growth_in_percent
  FROM 
    order_number_dec 
  JOIN 
    order_number_nov
  ON 
    order_number_dec.category = order_number_nov.category
  )

SELECT *
FROM main
ORDER BY growth_in_percent ASC
```

----

What is the best traffic source to get a session on thelook_ecommerce on Dec 2022?

```sql
SELECT 
  COUNT(traffic_source) AS total_traffic, traffic_source
  FROM
    (
    SELECT DISTINCT(session_id), traffic_source from sql-project-376612.thelook_ecommerce.events 
    WHERE DATE(created_at) BETWEEN '2022-12-01' AND '2022-12-31'
    )
GROUP BY traffic_source ORDER BY COUNT(traffic_source) DESC
```

----

Following the request above, Sheila wants to know the top 3 user’s first traffic sources that provide the most revenue in the past 3 months (current date 01 January 2023)

```sql
With events AS(
 SELECT
 traffic_source,
 user_id,
 created_at,
FROM
 `sql-project-376612.thelook_ecommerce.events`
 WHERE DATE(created_at) BETWEEN "2022-10-01" AND "2023-01-01"
 and user_id is not null)
,
Traffic_source AS(
SELECT *, 
 ROW_NUMBER () OVER (PARTITION BY user_id ORDER BY created_at) as rank
 FROM events 
ORDER BY 2,3)
,
first_traffic_source AS(
SELECT *
FROM traffic_source
WHERE rank = 1
)
,
transaction_traffic AS(
SELECT id, a.user_id, traffic_source, sale_price
FROM `sql-project-376612.thelook_ecommerce.order_items` a
join first_traffic_source b
ON a.user_id = b.user_id
WHERE status = "Complete" AND DATE(a.created_at) BETWEEN "2022-10-01" AND "2022-12-31"
)


SELECT traffic_source, sum(sale_price) AS sales
FROM transaction_traffic
GROUP BY 1
ORDER BY 2 DESC
```
