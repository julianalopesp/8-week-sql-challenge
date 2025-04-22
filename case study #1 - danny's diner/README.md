# 🍜 SQL Challenge: Danny's Diner

This project is part of the [8 Week SQL Challenge](https://8weeksqlchallenge.com/), created by Danny Ma.

Entity Relationship Diagram

![Danny's Diner](https://github.com/user-attachments/assets/4d2074e9-38bb-4277-ae2e-c25ebed718e5)

***

Questions
1. What is the total amount each customer spent at the restaurant?
```
SELECT
    s.customer_id,
    SUM(me.price)
FROM sales AS s
JOIN menu AS me
    ON me.product_id = s.product_id
GROUP BY s.customer_id
```

| customer_id | sum |
| ----------- | --- |
| B           | 74  |
| C           | 36  |
| A           | 76  |

---
2. How many days has each customer visited the restaurant?
```
SELECT
	s.customer_id,
    COUNT(DISTINCT s.order_date) AS days_visited
FROM sales s
GROUP BY s.customer_id
```
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

---

3. What was the first item from the menu purchased by each customer?
```
WITH ranked_sales AS (
	SELECT
    	s.customer_id,
        s.order_date,
        s.product_id,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date ASC) as rn
    FROM sales s
  )
    
SELECT 
	  rs.customer_id, 
    rs.order_date, 
    me.product_name
FROM ranked_sales AS rs
JOIN menu me ON me.product_id = rs.product_id
WHERE rs.rn = 1
```
| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | sushi        |
| B           | 2021-01-01 | curry        |
| C           | 2021-01-01 | ramen        |

---

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```
SELECT
    me.product_name,
    count(*) AS total_purchases
FROM sales s
JOIN menu me ON me.product_id = s.product_id
GROUP BY me.product_name
ORDER BY total_purchases DESC
LIMIT 1
```

| product_name | total_purchases |
| ------------ | --------------- |
| ramen        | 8               |

---
