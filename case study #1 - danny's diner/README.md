# 🍜 SQL Challenge: Danny's Diner

Este projeto faz parte do [8 Week SQL Challenge](https://8weeksqlchallenge.com/), criado por Danny Ma.

Entity Relationship Diagram

![Danny's Diner](https://github.com/user-attachments/assets/4d2074e9-38bb-4277-ae2e-c25ebed718e5)

***

### Questões

1. **Qual foi o valor total que cada cliente gastou no restaurante?**
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
2. **Quantos dias cada cliente visitou o restaurante?**
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

3. **Qual foi o primeiro item do cardápio comprado por cada cliente?**
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

4. **Qual é o item mais comprado do cardápio e quantas vezes ele foi comprado por todos os clientes?**
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

5. **Qual item foi o mais popular para cada cliente?**
```
WITH total_purchases AS (
  SELECT
	s.customer_id,
    	s.product_id,
    RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(product_id) DESC) AS rnk
    FROM sales s
  	GROUP BY s.customer_id, s.product_id
 )
    
SELECT
	tp.customer_id,
    	me.product_name
FROM total_purchases tp
JOIN menu me ON me.product_id = tp.product_id
WHERE rnk = 1
ORDER BY customer_id
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |
| B           | curry        |
| B           | ramen        |
| C           | ramen        |

---
6. **Qual item foi o primeiro comprado pelo cliente após se tornar um membro?**
```
WITH first_member_item AS (
  SELECT
  	s.customer_id,
  	s.order_date,
  	s.product_id,
  	mb.join_date,
  	ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date asc) AS rn
  	FROM sales s
  	JOIN members mb ON mb.customer_id = s.customer_id
  	WHERE s.order_date >= mb.join_date
  )
  
 SELECT
 	fm.customer_id,
    	me.product_name,
    	fm.order_date,
    	fm.join_date
 FROM first_member_item fm
 JOIN menu me ON me.product_id = fm.product_id
 WHERE rn = 1
```
| customer_id | product_name | order_date | join_date  |
| ----------- | ------------ | ---------- | ---------- |
| B           | sushi        | 2021-01-11 | 2021-01-09 |
| A           | curry        | 2021-01-07 | 2021-01-07 |

---

7. **Qual item foi comprado logo antes de o cliente se tornar um membro?**
```
WITH last_non_member_item AS (
   SELECT
   s.customer_id,
   s.product_id,
   s.order_date,
   RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date desc) AS rnk
   FROM sales s
   JOIN members mb ON mb.customer_id = s.customer_id
   WHERE s.order_date < mb.join_date
  )

SELECT 
	lnm.customer_id,
  	mn.product_name,
  	lnm.order_date
 FROM last_non_member_item lnm
 JOIN menu mn ON mn.product_id = lnm.product_id
 WHERE rnk = 1
```

| customer_id | product_name | order_date |
| ----------- | ------------ | ---------- |
| B           | sushi        | 2021-01-04 |
| A           | sushi        | 2021-01-01 |
| A           | curry        | 2021-01-01 |

---

8. **Qual é o total de itens e o valor gasto por cada membro antes de se tornarem membros?**
```
SELECT
 	s.customer_id,
 	COUNT(s.product_id) AS total_items,
 	SUM(mnu.price) AS total_spent
 FROM sales s
 JOIN members mb ON mb.customer_id = s.customer_id
 JOIN menu mnu ON mnu.product_id = s.product_id
 WHERE s.order_date < mb.join_date
 GROUP BY s.customer_id
 ```

| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| B           | 3           | 40          |
| A           | 2           | 25          |

---

9.  **Se cada $1 gasto equivale a 10 pontos e sushi tem um multiplicador de pontos de 2x – quantos pontos cada cliente teria?**
```
SELECT
 	s.customer_id,
    	SUM(
	      CASE
	      	WHEN mnu.product_name = 'sushi' THEN mnu.price * 10 * 2
	      ELSE mnu.price * 10
	      END
	      ) AS total_points
	      FROM sales s
	      JOIN menu mnu ON mnu.product_id = s.product_id
	      GROUP BY customer_id
```
| customer_id | total_points |
| ----------- | ------------ |
| B           | 940          |
| C           | 360          |
| A           | 860          |

---

10. **Na primeira semana após o cliente entrar no programa (incluindo a data de adesão), ele ganha 2x pontos em todos os itens, não apenas sushi – quantos pontos os clientes A e B teriam no final de janeiro?**
```
SELECT
	s.customer_id,
    	SUM(
	      mnu.price * 10 *
	      CASE
	      	WHEN s.order_date between mb.join_date AND mb.join_date + interval '6 days' THEN 2
	      	WHEN mnu.product_name = 'sushi' THEN 2
	      ELSE 1
	      END
	      ) AS total_points_january
	      FROM sales s
	      JOIN members mb ON mb.customer_id = s.customer_id
	      JOIN menu mnu ON mnu.product_id = s.product_id
	      WHERE DATE_TRUNC('month', s.order_date) = DATE '2021-01-01'
	      GROUP BY s.customer_id
```

| customer_id | total_points_january |
| ----------- | -------------------- |
| B           | 820                  |
| A           | 1370                 |
