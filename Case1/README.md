# Danny's Diner

[Case Study #1](https://8weeksqlchallenge.com/case-study-1/)

![Dannys](https://8weeksqlchallenge.com/images/case-study-designs/1.png)


1. What is the total amount each customer spent at the restaurant?
```sql
SELECT dannys_diner.sales.customer_id as "Customer", SUM(dannys_diner.menu.price) as "Total Spent"
FROM dannys_diner.sales
JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
GROUP BY dannys_diner.sales.customer_id
ORDER BY dannys_diner.sales.customer_id
```


| Customer | Total Spent |
| -------- | ----------- |
| A        | 76          |
| B        | 74          |
| C        | 36          |


---

2. How many days has each customer visited the restaurant?

```sql
SELECT dannys_diner.sales.customer_id as "Customer", COUNT(DISTINCT(dannys_diner.sales.order_date)) as "Number of days visited"
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
GROUP BY dannys_diner.sales.customer_id
```


| Customer | Number of days visited |
| -------- | ---------------------- |
| A        | 4                      |
| B        | 6                      |
| C        | 2                      |

---

3. What was the first item from the menu purchased by each customer?

```sql
WITH TABLE2 AS (SELECT dannys_diner.sales.customer_id, dannys_diner.sales.product_id
FROM dannys_diner.sales
INNER JOIN
(
	SELECT dannys_diner.sales.customer_id, MIN(dannys_diner.sales.order_date)
	FROM dannys_diner.sales
	INNER JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
	GROUP BY dannys_diner.sales.customer_id
) 
TABLE1
ON dannys_diner.sales.order_date = TABLE1.min AND dannys_diner.sales.customer_id = TABLE1.customer_id
)

SELECT TABLE2.customer_id, menu.product_name
FROM TABLE2
INNER JOIN dannys_diner.menu ON dannys_diner.menu.product_id = TABLE2.product_id
ORDER BY customer_id
```


| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |
| C           | ramen        |

---


4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT dannys_diner.menu.product_name, COUNT(dannys_diner.sales.order_date) as most_purchased
FROM dannys_diner.sales
JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY dannys_diner.menu.product_name
ORDER BY most_purchased DESC
LIMIT 1
```


| product_name | most_purchased |
| ------------ | -------------- |
| ramen        | 8              |

---


5. Which item was the most popular for each customer?


```sql
WITH TEMP AS (SELECT dannys_diner.sales.customer_id, dannys_diner.menu.product_name, COUNT(dannys_diner.sales.order_date) as most_purchased, DENSE_RANK() OVER   
    (PARTITION BY dannys_diner.sales.customer_id ORDER BY COUNT(sales.customer_id) DESC) AS Rank  
FROM dannys_diner.sales
JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY dannys_diner.sales.customer_id, dannys_diner.menu.product_name
ORDER BY most_purchased DESC)


SELECT customer_id, product_name, most_purchased
FROM TEMP
WHERE rank = 1
ORDER BY customer_id
```



| customer_id | product_name | most_purchased |
| ----------- | ------------ | -------------- |
| A           | ramen        | 3              |
| B           | curry        | 2              |
| B           | sushi        | 2              |
| B           | ramen        | 2              |
| C           | ramen        | 3              |

---

6. Which item was purchased first by the customer after they became a member?

```sql
WITH TEMP AS (SELECT dannys_diner.sales.customer_id, 
       dannys_diner.sales.order_date,
       dannys_diner.members.join_date,
       dannys_diner.sales.product_id,
       dannys_diner.menu.product_name,
       dannys_diner.menu.price,
		CASE WHEN (dannys_diner.members.customer_id = dannys_diner.sales.customer_id) AND (dannys_diner.sales.order_date >= dannys_diner.members.join_date)
        	THEN True
    		ELSE False
		END as "members?"
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
LEFT JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
)
, TEMP2 AS(SELECT TEMP.customer_id, min(TEMP.order_Date) as first_order
FROM TEMP
WHERE "members?" IS TRUE
GROUP BY TEMP.customer_id
          )


SELECT *
FROM (SELECT TEMP.customer_id, TEMP.product_name, TEMP.order_date FROM TEMP INNER JOIN TEMP2 ON (TEMP.customer_id = TEMP2.customer_id) AND (TEMP.order_date = TEMP2.first_order)) X
```


| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | curry        | 2021-01-07T00:00:00.000Z |
| B           | sushi        | 2021-01-11T00:00:00.000Z |

---

7. Which item was purchased just before the customer became a member?

```sql
WITH TEMP AS (SELECT dannys_diner.sales.customer_id, 
       dannys_diner.sales.order_date,
       dannys_diner.members.join_date,
       dannys_diner.sales.product_id,
       dannys_diner.menu.product_name,
       dannys_diner.menu.price,
		CASE WHEN (dannys_diner.members.customer_id = dannys_diner.sales.customer_id) AND (dannys_diner.sales.order_date >= dannys_diner.members.join_date)
        	THEN True
    		ELSE False
		END as "members?"
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
LEFT JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
)
, TEMP2 AS(SELECT TEMP.customer_id, max(TEMP.order_Date) as last_order_before_membership
FROM TEMP
WHERE "members?" IS False
GROUP BY TEMP.customer_id
          )


SELECT *
FROM (SELECT TEMP.customer_id, TEMP.product_name, TEMP.order_date FROM TEMP INNER JOIN TEMP2 ON (TEMP.customer_id = TEMP2.customer_id) AND (TEMP.order_date = TEMP2.last_order_before_membership)) X
```


| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | sushi        | 2021-01-01T00:00:00.000Z |
| A           | curry        | 2021-01-01T00:00:00.000Z |
| B           | sushi        | 2021-01-04T00:00:00.000Z |
| C           | ramen        | 2021-01-07T00:00:00.000Z |

---

8. What is the total items and amount spent for each member before they became a member?

```sql
WITH TEMP AS (SELECT dannys_diner.sales.customer_id, 
       dannys_diner.sales.order_date,
       dannys_diner.members.join_date,
       dannys_diner.sales.product_id,
       dannys_diner.menu.product_name,
       dannys_diner.menu.price,
		CASE WHEN (dannys_diner.members.customer_id = dannys_diner.sales.customer_id) AND (dannys_diner.sales.order_date >= dannys_diner.members.join_date)
        	THEN True
    		ELSE False
		END as "members?"
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
LEFT JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
)

SELECT TEMP.customer_id, SUM(TEMP.price) as total_spent_before_membership
FROM TEMP
WHERE "members?" IS False
GROUP BY TEMP.customer_id
```

| customer_id | total_spent_before_membership |
| ----------- | ----------------------------- |
| B           | 40                            |
| C           | 36                            |
| A           | 25                            |

---

9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
WITH TEMP AS (SELECT dannys_diner.sales.customer_id, 
       dannys_diner.sales.order_date,
       dannys_diner.members.join_date,
       dannys_diner.sales.product_id,
       dannys_diner.menu.product_name,
       dannys_diner.menu.price,
		CASE WHEN (dannys_diner.members.customer_id = dannys_diner.sales.customer_id) AND (dannys_diner.sales.order_date >= dannys_diner.members.join_date)
        	THEN True
    		ELSE False
		END as "members?",
        
        CASE WHEN (dannys_diner.menu.product_name = 'sushi')
        	THEN dannys_diner.menu.price * 10 * 2 
    		ELSE dannys_diner.menu.price * 10
		END as "points"
        
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
LEFT JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
)

SELECT TEMP.customer_id, SUM(TEMP.points) as total_points
FROM TEMP
GROUP BY TEMP.customer_id
```


| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

---

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
WITH TEMP AS (SELECT dannys_diner.sales.customer_id, 
       dannys_diner.sales.order_date,
       dannys_diner.members.join_date,
       dannys_diner.members.join_date + INTERVAL '6 day' AS bonus_deal_expires,
       dannys_diner.sales.product_id,
       dannys_diner.menu.product_name,
       dannys_diner.menu.price,
		CASE WHEN (dannys_diner.members.customer_id = dannys_diner.sales.customer_id) AND (dannys_diner.sales.order_date >= dannys_diner.members.join_date)
        	THEN True
    		ELSE False
		END as "members?"
        
FROM dannys_diner.sales
INNER JOIN dannys_diner.members ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
INNER JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
WHERE dannys_diner.sales.order_date <= '2021-01-31'
)


SELECT  TEMP.customer_id, SUM(
  CASE 
            WHEN (TEMP.order_date BETWEEN TEMP.join_date AND TEMP.bonus_deal_expires) THEN TEMP.price * 2 * 10
            WHEN (TEMP.product_name = 'sushi') THEN TEMP.price * 2 * 10
    		ELSE TEMP.price * 10
		END)
FROM TEMP
GROUP BY customer_id

```

| customer_id | sum  |
| ----------- | ---  |
| A           | 1370 |
| B           | 820  |

---


